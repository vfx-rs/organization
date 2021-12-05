GSOC proposals have to be about adding code to the project. Anything to do with things around the project such as documentation, testing, and etc should be reserved for a potential ASWF internship program.

Google Summer of Code
=====================

- Rewrite/refactor CPPMM (Mentors: Anders, Luke): CPPMM is our main tool to create the C and unsafe `*-sys` Rust bindings. It may be worth promoting to a product maintained by the ASWF, and used for other projects (including our own). The current implementation's design does not make it very easy to extend to C++ projects outside of what's currently being supported, so some work is needed to make it more extensible and support more valid C++ code.
- Write bindings for unclaimed projects (Mentors: Anders, Luke, Scott): Currently there are some bindings that are unclaimed (see `README.md` in this repo) that could use someone to build.
- Make C++ projects more Rust friendly (Mentors: Anders, Luke, Scott): There are some APIs that are not very Rust friendly, and also are considered difficult to support on the C++ side. The goal would be to work with the parent projects to update the problematic APIs.

ASWF Internship
===============

- CI support (Mentors: Scott): Currently we don't have CI set up, and we would like to have the following happen (in the rough order):
    1. Get tests passing on a modern Linux.
    2. Get the tests passing on CentOS 7.
    3. Get the tests passing on Windows 10 and latest MacOS.
    4. Add support for static checkers (such as Clippy, Cargo Audit, Cargo Deny, Miri, Rudra, etc).
    5. Add support for passing checks on MSRV, stable, beta, and nightly Rust.
- Project template (Mentors: Scott): Build a sample project that all projects should follow. This should contain the documentation (onboarding, license, contributors list, etc), structure for organizing the code (C bindings, Rust `*-sys` bindings, and Rust API), minimal CI, etc.
- Documentation (Mentors: Anders, Luke, Scott): Improve documentation in current bindings with example code (and if current Rust supports it, example code that passes when creating the docs), panic warnings, safety requirements, etc.
- Tests (Mentors: Anders, Luke, Scott): Add more tests to the currently bound wrappers. The goal of the tests are to test that our bindings perform the same in C and Rust as with the C++ code. We should not write tests that validate the underlying code unless that affects the C or Rust APIs.
