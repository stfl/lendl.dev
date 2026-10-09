+++
[extra]
hero_title = "Embedded Rust & Systems Engineering"
hero_subtitle = "Production Rust for devices in the field, and the CLIs, MCP servers and agents that operate them. 14 years from Yocto BSPs to railway OTA."
hero_cta = "Get in Touch"
hero_cv = "Download CV"
hero_cv_link = "https://github.com/stfl/cv/releases/latest/download/Stefan-Lendl-CV.pdf"

work_title = "Selected Work"
work_intro = "Shipped systems and open source, with the numbers that matter."
work_upstream = "Upstream contributions: OpenZFS, nixpkgs, meta‑rust, mcp‑server‑lib.el, copilot.el."

expertise_title = "Core Expertise"
expertise_intro = "Bringing modern Rust to embedded systems where reliability matters."

services_title = "Services"
services_intro = "Judgment first, then the hands-on work to carry it through"

experience_title = "Experience"

contact_title = "Let's Work Together"
contact_intro = "Whether you're building new embedded systems or modernizing existing platforms, I can help."
contact_location = "Vienna, Austria"
contact_availability = "Available in Vienna, partial travel, remote worldwide"
contact_github = "https://github.com/stfl"
contact_linkedin = "https://linkedin.com/in/stfl"

# Arrays must come after all scalar fields
[[extra.work]]
title = "Momentedge Clipper"
client = "Momentedge"
description = "Cuts event-triggered clips from a live ROS 2 MCAP recording while it is still being written. Reads the file only and never touches the recorder: 0.45 % of one core and 22 MiB on a Jetson Orin Nano."
tags = ["Rust", "ROS 2", "MCAP", "Jetson"]

[[extra.work]]
title = "Railway edge devices"
client = "ÖBB"
description = "Yocto LTS migration with an in-field upgrade path, signed A/B OTA updates with RAUC on ÖBB's PKI, and a Rust service that exports InfluxDB 3 measurements as Parquet over an unreliable link with zero data loss and a gapless completeness report."
tags = ["Rust", "Yocto", "RAUC", "PKI"]

[[extra.work]]
title = "sevDesk bookkeeping agent"
client = "Product in development"
description = "An AI agent that keeps a company's books through sevDesk. Its Rust CLI covers 327 API operations, 173 of them undocumented, rehearses every write as a dry run and demands a second flag before anything irreversible."
tags = ["Rust", "AI agents", "sevDesk"]
cta = "Want an AI bookkeeper for your sevDesk account? Talk to me"

[[extra.work]]
title = "typed-openapi"
client = "Open source"
description = "A typed Rust client and a clap command tree generated from one OpenAPI document. Every write prints the exact request and stops until --commit. Built for CLIs that an AI agent drives."
tags = ["Rust", "OpenAPI", "Code generation"]
link = "https://github.com/stfl/typed-openapi"

[[extra.work]]
title = "org-records-mcp"
client = "Open source"
description = "Turns a running Emacs into an MCP server over Org files: 34 tools, org-ql queries, records instead of text. In daily use with Claude Code."
tags = ["MCP", "Emacs Lisp", "AI agents"]
link = "https://github.com/stfl/org-records-mcp"

[[extra.work]]
title = "Proxmox Backup Server & OpenZFS"
client = "Proxmox"
description = "Backend features for Proxmox Backup Server in Rust with their ExtJS frontend, a kernel-module bug traced through ZFS mount handling and fixed upstream in OpenZFS, and Tier-3 enterprise support incidents across storage, networking and virtualization."
tags = ["Rust", "ZFS", "Proxmox", "Enterprise support"]
link = "https://github.com/openzfs/zfs/pull/15660"

[[extra.expertise]]
icon = "🦀"
title = "Embedded Rust"
description = "Modern, safe systems programming for embedded platforms. Production experience at ÖBB, Momentedge and Proxmox."

[[extra.expertise]]
icon = "🐧"
title = "Embedded Linux"
description = "Yocto/OpenEmbedded distributions for devices and NixOS for everything else: kernel configuration, BSP development, reproducible builds."

[[extra.expertise]]
icon = "🤖"
title = "AI Agents in Production"
description = "Agents that keep books, cut video and manage task systems, built on tools that refuse the irreversible by default."

[[extra.expertise]]
icon = "🏗️"
title = "Software Architecture"
description = "System design, requirements engineering, and modular framework development."

[[extra.services]]
title = "Rust Consulting & Engineering"
description = "Senior Rust for teams that run it in production or are about to: architecture, code review, mentoring, and hands-on delivery of the hard part. Embedded targets, Linux services, CLIs and protocol code."

[[extra.services]]
title = "Custom Linux with Yocto and NixOS"
description = "Yocto and OpenEmbedded distributions for devices, NixOS for servers and workstations: BSPs, kernel and driver work, signed A/B OTA updates with RAUC, PKI and code signing, device management."

[[extra.services]]
title = "Agent-Ready Tooling"
description = "CLIs and MCP servers an AI agent can drive without breaking things: typed clients from your OpenAPI spec, every write a dry run until --commit, one JSON envelope, honest exit codes."

[[extra.services]]
title = "Infrastructure & Reproducible Builds"
description = "NixOS hosts and development environments, Proxmox VE and Backup Server, CI/CD pipelines, and builds that produce the same artifact on a laptop, in CI and on the device."

[[extra.experience]]
title = "Senior Rust Engineer (Contract)"
company = "Momentedge"
description = "Momentedge Clipper (see Selected Work), with GitHub Actions releasing arm64 Debian packages for ROS 2 Humble and Jazzy."
period = "May 2026 – present"

[[extra.experience]]
title = "Senior Software Engineer (Contract)"
company = "ÖBB (Austrian Federal Railways)"
description = "Railway edge devices (see Selected Work), plus a Rust MQTT agent for ThingsBoard device management and requirements engineering with ÖBB stakeholders."
period = "Oct 2024 – present"

[[extra.experience]]
title = "Support Engineer (Contract)"
company = "Origina"
description = "Assessed Proxmox VE and Proxmox Backup Server for Origina's catalogue of independently supported enterprise software, mapping every functional area to what is supportable without vendor access and flagging the setups that pose a risk."
period = "Feb 2026 – Jun 2026"

[[extra.experience]]
title = "Embedded Software Architect & Technical Lead"
company = "3DataX (Client: TTTech)"
description = "Led the team and architected the C++ protocol bridging cloud commands to the vehicle bus on a custom Yocto Linux."
period = "May 2024 – Dec 2024"

[[extra.experience]]
title = "Software Engineer"
company = "Proxmox"
description = "Upstreamed an OpenZFS kernel module fix, built Proxmox Backup Server features in Rust, and resolved Tier-3 enterprise support incidents."
period = "Sep 2023 – Apr 2024"

[[extra.experience]]
title = "Software Engineer & Architect"
company = "pulswerk"
description = "Built a Django application from scratch and introduced Git and CI/CD; maintains and operates it under a freelance contract since 2022."
period = "Nov 2019 – present"

[[extra.experience]]
title = "Embedded Software Engineer"
company = "Mission Embedded"
description = "Yocto BSPs for i.MX, low-latency GStreamer pipelines for i.MX and Jetson, board bring-up, and a Rust configuration API."
period = "Oct 2014 – Oct 2019"

[[extra.experience]]
title = "Community Lead & Organizer"
company = "Rust Vienna Meetup"
description = "Grew Vienna's Rust community from 200 to 500+ members."
period = "Feb 2023 – Jun 2024"
+++
