# perl-critic-policy-prohibitleadingzeros

This distribution holds one Perl::Critic policy, `ProhibitLeadingZeros`. Perl reads an integer with a leading zero as octal. The policy reports such a literal. If the literal is a whole argument to a function that takes a file mode, for example `chmod`, the policy allows it. It is a fork of `Perl::Critic::Policy::Plicease::ProhibitLeadingZeros`.

## Layout

- `lib/Perl/Critic/Policy/ProhibitLeadingZeros.pm`: the policy and its POD.
- `t/main.t`: tables of snippets that the policy must report and snippets that it must allow.
- `.perlcriticrc` and `.perltidyrc`: the code standards.
- `dist.ini` and `prereqs.yml`: the build. The perl floor is 5.014, so every file keeps `use 5.014;`.
- `git-hooks/`: the hooks that the hub installs.

## Rules for work here

- Run the tests with `prove -l t`.
- Run the critic with `perlcritic --profile .perlcriticrc lib t`. It must print only `source OK` lines.
- Every `.pm` and `.t` file uses `use warnings FATAL => 'all';`. A `.t` file ends with `1;`.
- Never use `git commit --no-verify`. Never change a hook so that a commit passes.
- Turn off a critic policy only on purpose. Write the reason in a comment beside it and in the commit message.
- Do not change a test so that it passes. If a table entry is wrong, say why in the commit message.
- Add an entry to `Changes` for each change that a user can see.
- Write prose in plain English with short sentences.
