$schema: https://raw.githubusercontent.com/tabarra/txAdmin-recipe-schema/main/schema.json

name: Complete FivePD Server Stack
author: Custom
description: Full FivePD RP server with Qbox, vMenu, vehicle packs, Discord logs, admin permissions, and txAdmin integration

variables:
  steam_webapi:
    prompt: Steam Web API Key
    type: string

  discord_webhook:
    prompt: Discord Logging Webhook URL
    type: string

tasks:

# Create folder structure
  - action: ensure_dir
    path: ./resources/[qb]

  - action: ensure_dir
    path: ./resources/[standalone]

  - action: ensure_dir
    path: ./resources/[vehicles]

  - action: ensure_dir
    path: ./resources/[utils]

# Qbox Framework
  - action: download_github
    src: qbox-framework/qbox-core
    ref: main
    dest: ./resources/[qb]/qbox-core

  - action: download_github
    src: qbox-framework/qbox-jobs
    ref: main
    dest: ./resources/[qb]/qbox-jobs

# FivePD
  - action: download_github
    src: GitHubUser/FivePD
    ref: main
    dest: ./resources/[standalone]/fivepd

# vMenu Admin
  - action: download_github
    src: TomGrobbe/vMenu
    ref: master
    dest: ./resources/[standalone]/vmenu

# Dispatch / Callouts
  - action: download_github
    src: Project-Sloth/ps-dispatch
    ref: main
    dest: ./resources/[standalone]/pd-dispatch

# Vehicle packs
  - action: download_file
    url: https://github.com/citizenfx/cfx-server-data/archive/refs/heads/master.zip
    path: ./tmp/vehiclepacks.zip

# Discord Logging Scripts
  - action: download_github
    src: qbox-framework/qbox-logging
    ref: main
    dest: ./resources/[utils]/discord-logger

# Anti-Cheat (optional but recommended)
  - action: download_github
    src: guardian-ac/guardian
    ref: main
    dest: ./resources/[utils]/anti-cheat

# Database connection
  - action: connect_database

# Server.cfg setup
  - action: write_file
    file: ./server.cfg
    data: |
      endpoint_add_tcp "0.0.0.0:30120"
      endpoint_add_udp "0.0.0.0:30120"

      set steam_webApiKey "{{steam_webapi}}"

      sv_hostname "Qbox FivePD RP Server"
      sv_maxclients 64
      set sv_enforceGameBuild 2944
      setr txAdmin-menuEnabled true

      # ACE / Admin Permissions
      add_ace group.superadmin command allow
      add_ace group.admin command allow

      # Resources
      ensure qbox-core
      ensure qbox-jobs
      ensure fivepd
      ensure vmenu
      ensure pd-dispatch
      ensure discord-logger
      ensure anti-cheat

# Discord Webhook Configuration
  - action: write_file
    file: ./resources/[utils]/discord-logger/config.lua
    data: |
      Config = {}
      Config.Webhook = "{{discord_webhook}}"
