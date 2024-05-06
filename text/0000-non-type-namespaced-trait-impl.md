- Feature Name: `non_type_namespced_trait_impl`
- Start Date: 2024-5-6
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

Non-type-namespaced trait implementations, initialized as NTNTI, define a way for the implementor of a trait to exclude the trait's members from being looked up under the implementing type's namespace.

```rust
trait Foo {
    fn do_foo(&self);
}

#[non_type_namespaced]
impl Foo for i32 {
    fn do_foo(&self) {}
}

fn main() {
    let x = 4;
    // x.do_foo(); - error
    // i32::do_foo(&x); - error
    Foo::do_foo(&x); // ok
    <i32 as Foo>::do_foo(&x); // ok
}
```

# Motivation
[motivation]: #motivation

Why are we doing this? What use cases does it support? What is the expected outcome?

Sometimes, a trait may be appropriate to implement for a type, but the trait's members do not fit neatly into the type's existing API, or are likely to confuse users of the type.
Some examples from std:

- `impl<T: ?Sized> Clone for &T`: often called mistakenly as `my_ref.clone()` on a reference to a type that does not implement `Clone`.
- `impl<T> Clone for Rc<T>`: should be called as `Rc::clone(&rc)`, but the language permits non-conventional `rc.clone()`.
- `impl<T: ?Sized + 'static> Any for T`: `x.type_id()` is callable on smart pointers.

Allowing an implementor of a trait to exclude the trait's members from being looked up in the implementing type's namespace prevents method- and type-based lookup
from finding the members, while still allowing them to be invoked through the trait. Importantly, generic code can still rely on the trait's members being accessible,
while code dealing with concrete types must access the trait's members through a path mentioning the trait.

Making certain trait implementations NTNTI cleans up the implementing type's namespace if the implementor considers a different API more idiomatic, and in general gives more
control of a type's namespace to the type's author.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Explain the proposal as if it was already included in the language and you were teaching it to another Rust programmer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Rust programmers should *think* about the feature, and how it should impact the way they use Rust. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to existing Rust programmers and new Rust programmers.
- Discuss how this impacts the ability to read, understand, and maintain Rust code. Code is read and modified far more often than written; will the proposed feature make code easier to maintain?

For implementation-oriented RFCs (e.g. for compiler internals), this section should focus on how compiler contributors should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

**Note:** the `#[non_type_namespaced]` attribute syntax used here is illustrative only. While the author believes an attribute is the least diruptive way to add the necessary syntax,
she does not prefer a specific name or presence of an attribute.

By marking a trait implementation with the `#[non_type_namespaced]` attribute, the implementation becomes a _non-type-namespaced implementation_. The presence of a non-type-namespaced implementation
`#[non_type_namespaced] impl Trait for Type` prevents the members of `Trait` from being found by method lookup `x.member()` or path lookup `Type::member` in the type `Type`. There are no
other differences from a regular trait implementation. In particular, all code with access to both `Trait` and `Type` can witness that `Type` implements `Trait`, and the implementations of
`Trait`'s members for `Type` can be accessed through trait-based path lookup `Trait::member` and fully qualified `<Type as Trait>::member` syntax.

This is most useful for blanket implementations of a trait, where the usefulness of the trait's members to any specific type is unknown; and for implementations of a trait on
smart pointer types, where method syntax is not desirable.

For example, consider a 

```rust
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
- If this is a language proposal, could this be done in a library or macro instead? Does the proposed change make Rust code easier or harder to read, understand, and maintain?

# Prior art
[prior-art]: #prior-art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that rust sometimes intentionally diverges from common language features.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities
[future-possibilities]: #future-possibilities

Think about what the natural extension and evolution of your proposal would
be and how it would affect the language and project as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the project and language in your proposal.
Also consider how this all fits into the roadmap for the project
and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information.
