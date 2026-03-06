$schema: https://raw.githubusercontent.com/tabarra/txAdmin-recipe-schema/main/schema.json

name: QBCore RP Server Pack
author: Custom
description: Full QBCore server with vMenu, RP Essentials, vehicles, admin perms, Discord logs, and txAdmin integration

variables:
  steam_webapi:
    prompt: Steam Web API Key
    type: string

  discord_webhook:
    prompt: Discord Log Webhook URL
    type: string

tasks:

# Create directory structure
  - action: ensure_dir
    path: ./resources/[qb]

  - action: ensure_dir
    path: ./resources/[standalone]

  - action: ensure_dir
    path: ./resources/[vehicles]

# Install QBCore
  - action: download_github
    src: qbcore-framework/qb-core
    ref: main
    dest: ./resources/[qb]/qb-core

  - action: download_github
    src: qbcore-framework/qb-spawn
    ref: main
    dest: ./resources/[qb]/qb-spawn

  - action: download_github
    src: qbcore-framework/qb-inventory
    ref: main
    dest: ./resources/[qb]/qb-inventory

  - action: download_github
    src: qbcore-framework/qb-management
    ref: main
    dest: ./resources/[qb]/qb-management

  - action: download_github
    src: qbcore-framework/qb-adminmenu
    ref: main
    dest: ./resources/[qb]/qb-adminmenu

# Install vMenu
  - action: download_github
    src: TomGrobbe/vMenu
    ref: master
    dest: ./resources/[standalone]/vmenu

# RP Essentials
  - action: download_github
    src: Project-Sloth/ps-dispatch
    ref: main
    dest: ./resources/[standalone]/ps-dispatch

# Discord Logging
  - action: download_github
    src: qbcore-framework/qb-smallresources
    ref: main
    dest: ./resources/[qb]/qb-smallresources

# Vehicle Pack Loader Example
  - action: download_file
    url: https://github.com/citizenfx/cfx-server-data/archive/refs/heads/master.zip
    path: ./tmp/vehicles.zip

# Database
  - action: connect_database

# Server CFG setup
  - action: write_file
    file: ./server.cfg
    data: |
      endpoint_add_tcp "0.0.0.0:30120"
      endpoint_add_udp "0.0.0.0:30120"

      set steam_webApiKey "{{steam_webapi}}"

      sv_hostname "QBCore RP Server"
      sv_maxclients 64
      set sv_enforceGameBuild 2944

      setr txAdmin-menuEnabled true

      # Admin Permissions
      add_ace group.admin command allow
      add_ace group.superadmin command allow

      # Resources
      ensure qb-core
      ensure qb-spawn
      ensure qb-inventory
      ensure qb-management
      ensure qb-adminmenu
      ensure qb-smallresources
      ensure vmenu
      ensure ps-dispatch

# Discord Logging Config
  - action: write_file
    file: ./resources/[qb]/qb-smallresources/config.lua
    data: |
      Config = {}
      Config.DiscordWebhook = "{{discord_webhook}}"
