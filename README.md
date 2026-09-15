# NAME

Perl::Critic::Policy::ProhibitLeadingZeros - A leading zero is octal, which is right for a file mode and a bug anywhere else.

# VERSION

version 1.000

# Perl::Critic::Policy::ProhibitLeadingZeros

Perl reads an integer with a leading zero as octal, so `0032` is twenty-six.
Nobody writes that on purpose, except for a file mode, where octal is the
notation everyone reads:

```perl
chmod 0600, $key;           # a mode: fine
my $retries = 0010;         # eight, not ten
```

This policy reports the second and not the first.  A literal with leading zeros
is allowed when it is handed, whole, to a function or method that takes a mode;
everywhere else it is reported, and the fix is `oct('0010')` or no zeros.

This is a fork of
[Perl::Critic::Policy::Plicease::ProhibitLeadingZeros](https://metacpan.org/pod/Perl::Critic::Policy::Plicease::ProhibitLeadingZeros),
from [Perl-Critic-Plicease](https://github.com/uperl/Perl-Critic-Plicease) by
Graham Ollis.  That policy hard-codes `chmod` and `mkpath`; this one takes a
list, adds to it from configuration, and finds the call a literal belongs to by
walking the document rather than by counting siblings.

## PROHIBITED

```perl
my $z = 0032;
$mode & 0777;
my $perms = ( stat $file )[2] & 07777;
is( $mode, 0600, 'the key is private' );
chmod( ( stat $from )[2] & 07777, $to );   # a mask, not a mode
my %args = ( mode => 0755 );               # not yet handed to anything
use constant MODE => 0600;
```

## ALLOWED

```perl
umask 0022;
chmod 0600, $key;
chmod( 02750, $dir );
mkdir $dir, 0700 or die;
dbmopen( %cache, $file, 0600 );
sysopen( my $fh, $path, O_WRONLY | O_CREAT, 0600 );
mkpath( $dir, 1, 0700 );
make_path( $dir, { mode => 0711 } );
$path->chmod(0600);
```

And anything without a significant digit after its zeros, which is not octal by
accident: `0`, `00`, `0.5`, `0x1f`, `0b101`.

## CONFIGURATION

- `allow`

    Space separated list of functions and methods that may be handed a literal with
    leading zeros.  It **adds to** the built-in list rather than replacing it, so you
    name only your own:

    ```
    [ProhibitLeadingZeros]
    allow = Provisioner::Utils::write_pem Test::MockFile::new_dir
    ```

    The built-in list is:

    ```
    chmod dbmopen mkdir mkpath make_path sysopen umask
    ```

What a name matches:

- A name with no package, `chmod`

    Any call to a function or method of that name, however it is reached:
    `chmod(...)`, `CORE::chmod(...)`, `$path->chmod(...)` and
    `Some::Class->chmod(...)`.  This is what the built-in names are, which is
    how `$path->chmod(0600)` for [Path::Tiny](https://metacpan.org/pod/Path%3A%3ATiny) and `dir()->mkpath(1, 0700)`
    for [Path::Class](https://metacpan.org/pod/Path%3A%3AClass) are allowed.

- A name with a package, `Provisioner::Utils::write_pem`

    Only a call that names that package: `Provisioner::Utils::write_pem(...)`, or
    the class method `Provisioner::Utils->write_pem(...)`.  Not a bare
    `write_pem(...)`, and not `$object->write_pem(...)`, since the policy
    cannot know what package either one ends up in.

Where in the call the literal may be:

- In any argument

    There is no position.  `chmod(0600, $f)` and a mode as the fourth argument of
    `sysopen` are alike; so is a mode as the value in an anonymous hash or array of
    named arguments, `make_path( $dir, { mode => 0711 } )`, however deep.

- As the whole of that argument

    `0600` is allowed in `chmod 0600, $f`, and `07777` is not in
    `chmod( (stat $f)[2] & 07777, $t )`: next to any operator but a comma, a fat
    comma or a low-precedence `or`, `and` or `xor`, it is an operand in an
    expression rather than a mode.

- Of the nearest call

    In `chmod( foo(0600), $f )` the literal is `foo`'s argument, not `chmod`'s,
    and `foo` decides.

## CAVEATS

A mode has to be written where the call is.  One assigned to a variable or a
hash first, `my %opt = ( mode => 0755 )`, is not handed to anything the
policy can see, and is reported.

A mode chosen by a ternary, `chmod $dir ? 0755 : 0644, $f`, or combined
with another, `chmod 0666 & ~umask, $f`, is an operand and is reported too.
Write the combination out, or say `## no critic (ProhibitLeadingZeros)`.

A name with no package matches any method of that name on any object, since
there is no knowing what class an invocant is.  Name the package if that is too
broad.

## DIFFERENCES FROM THE ORIGINAL

What moving from `[Plicease::ProhibitLeadingZeros]` changes, besides `allow`:

- `umask 0022`, `mkdir`, `make_path`, `sysopen`, `dbmopen` and
`Some::Class->chmod(0600)` are allowed, where the original reported them.
- `mkpath` is allowed a mode however it is called.  The original allowed the
parenless form only with exactly three arguments, and the parenthesised form
only as a statement of its own, so not in `mkpath( $d, 1, 0700 ) or die`.
- A mask inside `chmod`'s arguments is reported.  The original allowed
`chmod( $m & 07777, $t )` and `chmod 0666 & ~umask, $f`.
- A `## no critic (Plicease::ProhibitLeadingZeros)` does not match this policy's
name, so an annotation that is still needed has to be renamed to
`## no critic (ProhibitLeadingZeros)`.

## SEE ALSO

[Perl::Critic::Policy::ValuesAndExpressions::ProhibitLeadingZeros](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3APolicy%3A%3AValuesAndExpressions%3A%3AProhibitLeadingZeros), the core
policy, which allows `chmod`, `dbmopen`, `mkdir`, `sysopen` and `umask` but
cannot be told about anything else.

## METHODS

### supported\_parameters

`allow`, the functions and methods that may take a literal with leading zeros,
added to the built-in list.

### initialize\_if\_enabled

Folds the built-in names back into whatever `allow` was configured with, so a
user's list adds to the defaults instead of replacing them.

### default\_severity

SEVERITY\_MEDIUM

### default\_themes

None, as in the original.

### applies\_to

PPI::Token::Number

### violates

Standard [Perl::Critic::Policy](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3APolicy) interface.  Returns a violation for a literal
with leading zeros, unless it is the whole of an argument to a call that
`allow` names.

# AUTHORS

Current Maintainers:

- George S. Baugh <teodesian@gmail.com>

Original author, of Perl::Critic::Policy::Plicease::ProhibitLeadingZeros:

- Graham Ollis <plicease@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2019-2024 by Graham Ollis, as
Perl::Critic::Policy::Plicease::ProhibitLeadingZeros in Perl-Critic-Plicease.

Modifications are copyright (c) 2026 by Troglodyne LLC.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
