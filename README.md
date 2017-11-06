# marshal

A [marshalling](https://en.wikipedia.org/wiki/Marshalling_(computer_science))
library for [rust](https://www.rust-lang.org).  Like
[pickle](https://docs.python.org/3/library/pickle.html) is for
[python](https://www.python.org/), but for [rust](https://www.rust-lang.org)

The lofty goal of marshal is to be **the** marshalling library for rust, in a
manner similar to how [serde](https://serde.rs/) is **the** serialization
library for rust.  That said, marshal is currently only in the planning stage.
Once we have a reasonable outline of what we want to accomplish, and how we want
to accomplish it, we'll move forwards with actually implementing marshal.

Planning is done via marshal's [GitHub
issues](https://github.com/cfkaran2/marshal/issues).  Whenever an issue is
wrapped up, a summary of what was discussed is moved to
[docs/PLANNING.md](docs/PLANNING.md).  This makes it easier for new
collaborators to come up to speed on marshal quickly, and helps old contributors
jog their memories.

## How to Help

Help us plan on how to create marshal.  Issues that are marked 'RFC' are issues
that will affect the direction that marshal takes ('help wanted' and other tags
are reserved for the actual implementation of what is planned).  Then read
[docs/PLANNING.md](docs/PLANNING.md); if you see something you disagree with,
either join the discussion in one of the referenced issues, or if none of the
issues apply, start a new issue.

# License

Marshal is licensed under the [Apache License, Version
2.0](http://www.apache.org/licenses/LICENSE-2.0).  A copy of the full license
text can be found in the [LICENSE](LICENSE) file.

# Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in Marshal by you, as defined in the Apache-2.0 license, shall be
licensed as above, without any additional terms or conditions.
