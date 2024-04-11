# HLSL Design Considerations

When designing and proposing a feature for HLSL there are some design
considerations that should be taken into account.

## Design Philosophy

There are a lot of examples of different approaches to programming language
design. Taking inspiration, and some direct quotes, from [The Zen of
Python](https://peps.python.org/pep-0020/),below is a list of guiding principles
for HLSL's evolution:

### Explicit is better than implicit.

Our users are highly technical. They care deeply about both the performance and
portability of the HLSL they write and maintain.

### Prioritize simplicity.

Correlations between code complexity and bugs are widely accepted. Complex code
is also known to be more difficult to test, maintain, and extend. While a user
may not care if the compiler code is simple, they do care if a compiler is
buggy and slow to evolve.

We should aim to solve problems with simple solutions, but some problems require
a complex solution. In those cases we should strive to avoid complications in
both design and implementation; keeping design as simple as possible.

### Readability counts.

Code is an evolving construct. As the technologies, people, and social
structures around the code change code will need to evolve and change itself.
The absence of readability becomes a barrier to evolution.

### Special cases aren't special enough to break the rules.

The rules of the programming language need to apply everywhere. The rules of our
design process need to apply everywhere. The rules of our coding standards need
to apply everywhere. Rules can change, but the can't be broken.

### If you don't have time to do it right, you must have time to do it again.

We should not rush design decisions for artificial reasons. When we ship a
feature in HLSL our users will begin to depend on it which makes it hard for us
to change it in the future without disrupting our users. For that reason we owe
it to our users to be deliberate and thoughtful in our design so that we have
the best chances of getting it right the first time.

### We will make mistakes, we can't be afraid to fix them.

Despite the point above, we are human. We will make mistakes. We can't be afraid
to fix them even when it is scary or might cause unavoidable short term pain.

### It is better to fail gracefully than succeed magically.

There is nothing more frustrating than something not working as intended and
providing bad feedback to the user. The Zen of Python says, "In the face of
ambiguity, refuse the temptation to guess". We should prefer failing gracefully
and giving actionable feedback to our users rather than trying to make our tools
or language "guess" what the user intended because guessing wrong is worse.

### Embrace idioms, standards, and common patterns.

The default assumption should be to seek alignment. Whether that is aligning
with the open source communities we work in, aligning with industry and
international standards, or aligning with commonly acknowledged practices and
patterns. We should not be different for the sake of being different, we should
be different when there are compelling reasons to be different.

## Style Conventions

HLSL's built-in types and methods should conform to a consistent coding style.

* Data types, methods and built-in functions should all be `CamelCase`.
* Namespaces and keywords are lowercase, `_` separated.
* Vulkan-specific functionality should be added to the `vk` namespace.
* Microsoft-style attributes interchangably use `CamelCase` and `_` separation,
  but should prefer `CamelCase`
* System Value semantics are case insensitive, should be specified `CamelCase`
  and prefixed `SV_`.

## Versioning

All features should consider how users can adapt to the presence or absence of
support. HLSL has two primary revisioning axis: language version and runtime
version. How the two versions interact is not always a clean and easy line to
see.

### Language Changes

> Language versioning changes the core language of HLSL: syntax, grammar,
> semantics, etc.

HLSL identifies language versions by the year of release (2015, 2016, 2017,
2018, 2021, ...), and future language versions have placeholder years (202x,
202y, ...).

Most language features do not require underlying runtime features so they can be
exposed in HLSL regardless of runtime targeting.

Some HLSL language features are _strictly additive_, and may be retroactively
enabled in older language modes. See the section below on "Exposing versioning"
for more information about retroactively exposing features.

### Runtime Changes

> Runtime versioning changes the library functionality of the language: data
> types, methods, etc.

HLSL's supported runtimes are DirectX and Vulkan. For DirectX versioning of HLSL
is broken down by Shader Model and DXIL version, and for Vulkan versioning is
broken down by Vulkan and SPIR-V version.

When a new runtime version is released and no previous HLSL compilers have
supported it, the feature can be added dependent only on targeting the new
runtime version. When a feature is added to a runtime version that has
previously been supported by a compiler release, the feature should be treated
as a retroactive addition. See the section below on "Exposing versioning" for
more information about retroactively exposing features.

### Exposing versioning

HLSL language and target runtime versions are exposed in the HLSL preprocessor
via the built-in preprocessor macros described below:

* **`__HLSL_VERSION`** - Integer value for the HLSL language version. Unreleased
  or experimental language versions are defined as a number larger than the
  highest released version.
* **`__SHADER_TARGET_STAGE`** - Integer value corresponding to the shader stage.
  Shader stage. The shader stage values are exposed as
  `__SHADER_STAGE_**STAGE**` (i.e. `__SHADER_STAGE_VERTEX`,
  `__SHADER_STAGE_PIXEL`, ...)
* **`__SHADER_TARGET_MAJOR`** - Major version for Shader Model target.
* **`__SHADER_TARGET_MINOR`** - Minor version for Shader Model target.

If these macros are not sufficient for a given feature new macros or other
mechanisms should be added as appropriate for the feature to enable developers
to know if a given compiler release supports the required feature(s).

For features that are added retroactively to an already final runtime or
language version, a `__has_feature` check should be added to allow the user to
query for feature support in the preprocessor instead of forcing the user to
check compiler versions explicitly. The Clang feature checking macros are
documented
[here](https://clang.llvm.org/docs/LanguageExtensions.html#feature-checking-macros).
The `__has_feature` macro is known to work in DXC, and should be used.
