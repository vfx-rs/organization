

# Rust Bindings Working Group

The Rust bindings working group is dedicated to creating a foundation for C and Rust bindings for C++ libraries used by the media and entertainment industry.

Goals of the WG are:

- To collaborate with ASWF communities to create (or facilitate the creation of) C and Rust bindings which are consistent across projects.
- Create tooling to help build the C and Rust bindings.
- Define a set of conventions and rules to create a consistent and ergonomic C and Rust interface for the wrapped libraries.
- To have the Rust bindings easily accessible through Rust's crates.io repository, and to give ownership of those bindings to the ASWF.

Non-goals of the WG are:

- Re-implementing current C++ libraries in Rust.

The TAC member sponsor of this working group is Sean Looper

## Deliverables

- Bind Imath and OpenEXR with C and Rust bindings
- Create a framework for binding the other libraries

## Communication

This WG communicates on the following channels:

- [vfx-rs organization repository on GitHub](https://github.com/vfx-rs/organization/issues)
- #Rust on ASWF slack.

## Meetings

See the [ASWF public calendar](https://lists.aswf.io/calendar). This WG meets the first Friday of every month at 12 pm Pacific Time.

_provide the Zoom/conference call details_

## In-person meetings

_list if applicable, or skip if not_

## Meeting notes

Meeting notes, recordings, and any presentations made during WG meetings are available [here](meetings).

## Tracked Libraries

| Name                  | Status      | Repository                                               | Binding Development                    | Crate                                        |
| --------------------- | ----------- | -------------------------------------------------------- | -------------------------------------- | -------------------------------------------- |
| Alembic               | Not Started | https://github.com/alembic/alembic                       |                                        | https://crates.io/crates/alembic             |
| Imath                 | Not Started | https://github.com/AcademySoftwareFoundation/Imath       |                                        | https://crates.io/crates/imath               |
| MaterialX             | Not Started | https://github.com/materialx/MaterialX                   |                                        | https://crates.io/crates/materialx           |
| Open Shading Language | Not Started | https://github.com/imageworks/OpenShadingLanguage        |                                        | https://crates.io/crates/openshadinglanguage |
| OpenColorIO           | Not Started | https://github.com/AcademySoftwareFoundation/OpenColorIO |                                        | https://crates.io/crates/opencolorio         |
| OpenEXR               | In Progress | https://github.com/AcademySoftwareFoundation/openexr     | https://github.com/vfx-rs/openexr-bind | https://crates.io/crates/openexr             |
| OpenImageIO           | In Progress | https://github.com/OpenImageIO/oiio                      | https://github.com/vfx-rs/oiio-bind    | https://crates.io/crates/openimageio         |
| OpenSubdiv            | Not Started | https://github.com/PixarAnimationStudios/OpenSubdiv      |                                        | https://crates.io/crates/opensubdiv          |
| OpenTimelineIO        | Not Started | https://github.com/PixarAnimationStudios/OpenTimelineIO  |                                        | https://crates.io/crates/opentimelineio      |
| OpenVDB               | Not Started | https://github.com/AcademySoftwareFoundation/openvdb     |                                        | https://crates.io/crates/openvdb             |
| Ptex                  | In Progress | https://github.com/wdas/ptex                             | https://github.com/vfx-rs/ptex-bind    | https://crates.io/crates/ptex                |
| SeExpr                | Not Started | https://github.com/wdas/SeExpr                           |                                        | https://crates.io/crates/seexpr              |
| USD                   | In Progress | https://github.com/PixarAnimationStudios/USD             | https://github.com/vfx-rs/usd-bind     | https://crates.io/crates/usd                 |
