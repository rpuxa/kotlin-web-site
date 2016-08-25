---
layout: docs-page
title: Videos
docs_nav_id: videos
---


<div class="g-grid">
    <div class="g-3">
        <div class="video-gallery" id="video-gallery"></div>
    </div>

    <article role="main" class="page-content g-9">
        <div id="video-player"></div>
        <div id="video-description" class="video-gallery-description"></div>
    </article>
</div>

<script>
    require(['page/videos'], function (page) {
        $.getJSON('/data/videos.json', function (videos) {
            page({
                elem: document.getElementById('video-gallery'),
                playerElem: document.getElementById('video-player'),
                descriptionElem: document.getElementById('video-description'),
                videos: videos
            });
        })

    });
</script>