---
layout: post
title: "Using Procs as Filters for RSpec Metadata"
date: 2024-10-02 17:33:06 +0530
categories: ruby
---

Ever wondered why we have to add RSpec Helpers multiple times for each spec type? There must be a more idiomatic way to do this.

```ruby
RSpec.configure do |config|
  config.include RequestSpecHelper, type: :controller
  config.include RequestSpecHelper, type: :service
end
```

I suspected others might have the same question, so I put on my detective hat and jumped into RSpec core [repository](https://github.com/rspec/rspec-core) issues. Behold, there was already an issue descrbing exactly this. ["#2890 Allow array as value for type key in Configuration#include"](https://github.com/rspec/rspec-core/issues/2890).

Here, Phil [@pirj](https://github.com/pirj) explained in this [comment](https://github.com/rspec/rspec-core/issues/2890#issuecomment-825886660) that introducing special handling for `type`, especially for `config.include`, risks breaking existing code that relies on standard behavior. And desired behaviour can easily be acheived with current rspec metadata options, which accepts `Proc` as a value.

```ruby
RSpec.configure do |config|
  config.include RequestSpecHelper, type: ->(type) { [:request, :component].include?(type) }
end
```

### Small contributions towards docs

After learning this, I decided to contribute back by creating a follow-up PR that adds an example of using Procs with `config.include`. You can check it out here: [PR #3112](https://github.com/rspec/rspec-core/pull/3112).

A special shoutout to RSpec maintainer [@JonRowe](https://github.com/JonRowe) for the quick review and merge. It's always great to see maintainers so responsive to contributions! 🎉

### Learnings

This whole experience was a valuable learning opportunity, seeing how maintainers balance shiny new features and refactors with backward compatibility. More importantly, it shows that just using Ruby can take us a long way.
