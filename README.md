Reflecting on My GSoC 2024 Experience with VideoLAN
===================================================

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*lH_tn5UpRYQaHq_SW5EEZg.png)

Introduction
============

My journey with VideoLAN began with an interest for open-source software and a desire to contribute to something impactful. I have been using VLC media player for as long as I can remember. So when I was browsing through the organisations announced for GSoC, I knew it was the only organization I wanted to apply for. This blog post reflects my GSoC experience, the challenges I faced, the skills I developed, and the invaluable lessons I learned that I hope will benefit the community.

Getting Started
===============

Initially, I went through the GSoC ideas shared by VideoLAN. However, I was particularly interested in working on some personal feature ideas I’ve been wanting to see in VLC Player. My ideas included adding support for pinch-to-zoom and integrating the OpenSubtitles API to fetch subtitles for media files that users might lack in the desktop app. I reached out to the maintainers regarding these ideas. Unfortunately, I learned that the first idea would be too challenging and outside the scope of GSoC, while the latter was already implemented.

Consequently, I revisited the list of ideas, and one that especially caught my interest was the “Port of Remote Access client from VLC or Android to the desktop app.” I’ll expand on this in more detail later in the blog.

To prepare, I started exploring every available article on the VideoLAN wiki for developers and contributors, aiming to familiarize myself with the codebase. The most challenging part of the entire project was the beginning. I struggled significantly with compiling the project and setting up the IRC chat bouncer. At the time, the project was undergoing continuous changes in dependencies, and I encountered errors related to missing Qt libraries. Despite going through the documentation, it didn’t help since the changes were recent and not yet reflected in the build systems.

After several days of being stuck, I finally managed to compile the project with the help of my project mentors, Alaric Senat and Nicolas Pomepuy. They provided me with a list of specific Qt packages that I needed to install, which resolved the compilation errors and allowed me to run the project. This was a significant milestone.

Given that the VLC codebase is vast and this was my first time working on a real-world project of such magnitude, I had to ask my mentors to direct me to the relevant parts of the codebase so I could start reading and understanding them.

Application Process
===================

I read the blogs written by past GSoC students who were selected by VideoLAN and shared their experiences and proposals. These were quite informative and helpful. I started writing my proposal 10 days before the deadline to allow time for validation by the mentors and to incorporate any suggestions or corrections they provided.

In the period between submitting my proposal and the start of the coding phase, I spent a significant amount of time understanding the codebase and getting my doubts clarified.

Project Details
===============

The project involved porting the VLC for Android remote access feature to VLC Desktop. VLC already had a web interface, but it was outdated and lacked many new elements introduced in VLC4, such as the medialibrary, which scans storage, lists all media files, and acts as a central hub for browsing, searching, and categorizing media. The existing interface was outdated in terms of the technologies used. The new remote access client was built using Vue for the frontend and WebSockets for real-time playback updates, offering enhanced security. In contrast, the old client relied on jQuery and HTTP requests for playback updates.

![Legacy web interface [Old]](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*fnEigNlBBvli_CXSMghhvQ.png)

The first step was to extract the Vue frontend code and place it in a [dedicated repository](https://code.videolan.org/videolan/remoteaccess) so it could be maintained separately and eventually ported to all other VLC platforms. This task was handled by my mentor, Nicolas (VLC Android Lead), who had developed this client for the Android app.

![Remote Access interface [New]](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*f8cPc3t6gXIqO42mdcKM8w.png)

VLC already had an existing web server written in C, with its APIs exposed using Lua. Most of the APIs for playback were already exposed through Lua for the legacy web interface, which could be directly used for the new client. However, the Lua APIs for the media library had not yet been exposed. So, I began working on exposing the Lua APIs for the underlying media library C APIs.

[Merge Request Link](https://code.videolan.org/videolan/vlc/-/merge_requests/5891)

I also created abstractions for the `setFavorite` APIs and `isFavorite` attributes in C, as I noticed they were missing when I tried to expose their Lua APIs.

[Merge Request Link](https://code.videolan.org/videolan/vlc/-/merge_requests/5869)

We now have Lua APIs for accessing the necessary medialibrary metadata. However, the client’s accepted data format differs from what we receive from the medialibrary. To address this, I created wrappers to format the raw data and ensure compatibility with the client. Additionally, The interface offers functionality to browser the server’s local and network storages so I wrote a Lua module to browse system directories and retrieve metadata for files and folders, including size, the number of media files and subfolders, and thumbnails.

The remote interface relied on WebSockets for real-time playback updates, but since the server lacked support for both WebSockets and long polling, I had to use short polling instead. This involved sending HTTP requests continuously at a fixed interval of 500ms, which is quite inefficient and should be addressed in the future. During development and testing, I also reported several bugs related to UI components, backend functions, and build scripts.

Work Done
=========

I have implemented the following 25 [API handlers](https://code.videolan.org/mohak2003/vlc/-/blob/remoteserver/share/lua/intf/remote-access.lua?ref_type=heads) and their [callbacks](https://code.videolan.org/mohak2003/vlc/-/blob/remoteserver/share/lua/intf/modules/callbacks.lua?ref_type=heads) :

*   `**/translation**`: Returns translation data.
*   `**/video-list**`: Returns a list of videos; optional grouping and folder parameters.
*   `**/artwork**`: Returns thumbnails and covers; accepts media-related parameters.
*   `**/history**`: Returns user history.
*   `**/album-list**`: Returns a list of all albums.
*   `**/artist-list**`: Returns a list of all artists.
*   `**/track-list**`: Returns a list of all tracks.
*   `**/genre-list**`: Returns a list of all genres.
*   `**/playlist-list**`: Returns a list of all playlists.
*   `**/playlist**`: Returns details of a specific playlist; requires playlist ID.
*   `**/artist**`: Returns details of a specific artist; requires artist ID.
*   `**/album**`: Returns details of a specific album; requires album ID.
*   `**/genre**`: Returns details of a specific genre; requires genre ID.
*   `**/play**`: Plays single/multiple media files; optional media and playback parameters.
*   `**/storage-list**`: Returns a list of storage devices or locations.
*   `**/browse-list**`: Returns a list of items in a directory; requires folder path.
*   `**/icon**`: Returns an icon; accepts asset-related parameters.
*   `**/favorite**`: Sets/Unset a media item as favorite; requires media ID and type.
*   `**/favorite-list**`: Returns a list of all favorite items.
*   `**/longpolling**`: Returns real-time updates or events.
*   `**/search**`: Returns search results; requires a query.
*   `**/playback-event**`: Used to execute playback related tasks; accepts event details.
*   `**/resume-playback**`: Resumes last played media.
*   `**/stream-list**`: Returns a list of past streams.
*   `**/playlist-add**`: Used to add a media item in a playlist; requires media and playlist details.

Limitations / Future work
=========================

I encountered a few challenges that still need to be addressed for project completion:

1.  **Translation strings missing**: some translation strings are not yet available but will be added during the time of release.
2.  **No Long Polling or WebSocket Support:** The VLC Desktop server currently lacks support for asynchronous HTTP request handling, which is necessary for long polling or WebSockets. As a workaround, I had to implement short polling on the server, which is less efficient compared to the former methods.
3.  **Setter APIs Not Allowed for Security Concerns:** Lua APIs are not highly secure, so allowing remote access to modify local files is unsafe. This limitation prevents the implementation of setter APIs on the server. Permissions management needs to implemented similar to the android port. Here is some [reference](https://code.videolan.org/videolan/vlc/-/merge_requests/5891#note_450006) for this issue.
4.  **Server Doesn’t Generate Video-Folder Thumbnails:** Thumbnails for video folders are currently generated separately using QT libraries. The logic for this needs to be imported into Lua.
5.  **Missing Lua APIs**: The Remote Interface supports more VLC functionalities, but certain APIs, such as Sleep, are not currently exposed via Lua.
6.  **No Zipping Functionality for Bulk Downloads:** While media files can be downloaded using the remote web interface, bulk downloads are problematic. The server needs to zip the files before sending them, as most browsers block multiple downloads.
7.  **No Support for Multi-part POST Requests**: The Remote Interface uses multi-part POST requests to upload large files, but the existing HTTP 1.1 server does not support this functionality.
8.  **No Support for Network Storages:** Lua APIs for accessing network storages have not yet been exposed and need to be developed in the future.
9.  **Additional handlers can be added**: This is the complete [list](https://code.videolan.org/videolan/remoteaccess/-/blob/main/src/plugins/api.js?ref_type=heads) of requests the interface can make to the server. Some lower-priority ones are yet to be implemented.
10.  **Build System Update**: The code for the remote access interface needs to be fetched from its dedicated repository during the compilation process. To achieve this, we need to modify the Meson build configuration accordingly.

Learnings
=========

I got hands-on experience with C programming and OOP, which I had previously only studied theoretically. I encountered several errors during coding, and my mentor, Alaric, suggested learning gdb, which was very useful for debugging and fixing issues. Additionally, I explored VueJS, a framework I hadn’t worked with previously. The project pushed me to think critically and focus on writing clean, efficient code.

Beyond technical skills, I learned the importance of effective communication, especially in a remote, distributed team. Regular communication with my mentors was essential in keeping the project on track, avoiding blockers, and refining my work.

Mentorship and Community
========================

My mentors were not only technically proficient but also very supportive. They offered constructive criticism and guided me through complex challenges. Contributing to such a respected project was a privilege, and I felt a strong sense of belonging in the open-source community.

Conclusion
==========

I am deeply grateful to my mentors and the VideoLAN community for providing this opportunity. If you have any questions or would like to learn more about my project or GSoC, please feel free to reach out to me via [email](http://mohak2003@gmail.com).
