# vive-coding-playground

**English** | [한국어](README.ko.md)

A home for small projects built by vibe coding.

Ideas are written down first, and the ones worth building are implemented together with an AI coding agent (Claude Code). Each finished project lives in its own repository and is referenced here as a git submodule.

You can browse everything built so far at the [Vibe Museum](https://vive-museum.yeonhyuk.me).

| [Warehouse Tycoon](https://warehouse-tycoon.yeonhyuk.me/en/) | [Vibe Museum](https://vive-museum.yeonhyuk.me) |
|---|---|
| ![Warehouse Tycoon](https://raw.githubusercontent.com/BranKein/vive-warehouse-tycoon/main/deploy/thumbnail.jpg) | ![Vibe Museum](https://raw.githubusercontent.com/BranKein/vive-museum/main/docs/screenshots/pinboard-en.jpg) |

## How it works

1. **Write the idea down**: ideas and requirements go into [`ideas/`](ideas/) as documents.
2. **Build it**: an idea chosen for building is implemented in its own repository. The agent plans the work and splits it up.
3. **Link it**: the finished project is added here as a submodule.

## Projects

| Project | Status | Description |
|---|---|---|
| [warehouse-tycoon](https://github.com/BranKein/vive-warehouse-tycoon) | Done | A web-based 3D idle tycoon game about warehouses. [Play](https://warehouse-tycoon.yeonhyuk.me/en/) |
| infra (private) | Done | Terraform code that puts the projects on AWS |
| [home](https://github.com/BranKein/vive-home) | In progress | About the maker and the projects. [yeonhyuk.me](https://yeonhyuk.me) |
| [museum](https://github.com/BranKein/vive-museum) | In progress | A museum site that shows this repository's projects in four different halls. [Visit](https://vive-museum.yeonhyuk.me) |

## Ideas

The idea documents are written in Korean.

| Idea | Status | Description |
|---|---|---|
| [Home site for the root domain](ideas/home-site.md) | In progress | The front page of `yeonhyuk.me`, introducing the maker and the projects |
| [iKeyboard & iMouse](ideas/ikeyboard-imouse.md) | Idea | An experiment that reads hand movements on a desk with a camera and turns them into virtual keyboard and mouse input |
| [Periodic coding problems with subscriptions](ideas/periodic-coding-problems.md) | Idea | Generates coding and SQL problems on a schedule and emails subscribers |
| [Terraform infrastructure for AWS](ideas/aws-terraform-infra.md) | Done | Keeps the Terraform code that puts the other projects on AWS in one place |
| [Warehouse tycoon game](ideas/warehouse-tycoon.md) | Done | A web-based 3D idle tycoon game about warehouses |
| [Project museum site](ideas/project-museum-site.md) | In progress | A site that exhibits the projects like a museum and takes ideas from visitors |

The full list and the meaning of each status are in [ideas/README.md](ideas/README.md).

## Repository layout

```
.
├── ideas/       # idea and requirement documents
├── docs/        # shared docs: deployment, SEO, ads
├── projects/    # individual projects (git submodules)
├── AGENT.md     # working rules the AI agent follows
└── CLAUDE.md    # entry point for Claude Code
```
