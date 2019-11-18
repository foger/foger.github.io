---
layout: default
title: About me
permalink: /about.html
---

# About me

<p align="center">
    <img class="avatar" src="">
</p>

Hello. My name is Sergey Shutko. I'm interested in `DevOps`.<br/>
View this repository on <a href="{{ site.github.repository_url }}">GitHub</a>

<script>
    var avatar = document.getElementsByClassName('avatar')[0]
    console.log(avatar)
    fetch('https2://api.github.com/users/foger')
    .then(response => response.json())
    .then(data => {
        avatar.src = data['avatar_url']
    })
    .catch(error => {
        avatar.src = "assets/img/avatar.png"
        console.error(error)
    })
</script>
