# What is spec/lib/activitypub/parser/status_parser_spec.rb?

## Overview

`spec/lib/activitypub/parser/status_parser_spec.rb` is a **RSpec test file** that contains unit tests for the `ActivityPub::Parser::StatusParser` class.

## Location
```
mastodon/
  └── spec/                           # Test directory
      └── lib/                        # Tests for lib/ code
          └── activitypub/            # ActivityPub-related tests
              └── parser/             # Parser tests
                  └── status_parser_spec.rb  # ← This file
```

## What Does It Test?

The file tests the `ActivityPub::Parser::StatusParser` class located at:
```
app/lib/activitypub/parser/status_parser.rb
```

## Purpose of the StatusParser Class

The `StatusParser` class is responsible for:
1. **Parsing ActivityPub objects** - Converting JSON ActivityPub data into Mastodon-readable format
2. **Extracting content** - Getting text, title, summary, URLs from ActivityPub objects
3. **Processing text** - Formatting content for display in Mastodon's UI
4. **Handling different object types**:
   - `Note` - Regular social media posts (Mastodon, Pleroma, etc.)
   - `Article` - Long-form content (Lemmy, Friendica, WriteFreely)
   - `Page` - Static pages (WordPress, etc.)
   - `Event` - Event announcements
   - `Video`, `Audio`, `Image` - Media content

## Test Structure

### Existing Tests (Before Our Changes)
```ruby
RSpec.describe ActivityPub::Parser::StatusParser do
  # Tests basic status parsing
  it 'correctly parses status'
  
  # Tests likes collection handling
  context 'when the likes collection is not inlined'
  
  # Tests quote policy feature
  describe '#quote_policy' do
    # Multiple test cases for quote permissions
  end
end
```

### New Tests We Added
```ruby
describe '#processed_text' do
  # Tests for the processed_text method that formats content
  
  context 'when object is a Note'
    # Baseline test - regular posts
  
  context 'when object is an Article'
    # NEW: Tests Article with body content
  
  context 'when object is a Page'
    # NEW: Tests Page with body content
  
  context 'when object is an Event'
    # NEW: Tests Event with body content
  
  context 'when Article has no body content'
    # NEW: Edge case test
end
```

## What We Changed

### The Code Change (1 line)
In `app/lib/activitypub/parser/status_parser.rb`:
```ruby
def processed_text
  return text || '' unless converted_object_type?

  [
    title.presence && "<h2>#{title}</h2>",
    spoiler_text.presence,
    text.presence,  # ← ADDED THIS LINE
    linkify(url || uri),
  ].compact.join("\n\n")
end
```

### The Test Changes (103 lines)
We added 5 comprehensive test cases to verify the new behavior works correctly.

## Why This Test File Matters

### Before Our Change
```ruby
# Article from Lemmy/Friendica
article = {
  type: 'Article',
  name: 'My Blog Post',
  content: 'This is the full article text...',
  url: 'https://lemmy.example.com/post/123'
}

# Mastodon displayed:
# <h2>My Blog Post</h2>
# https://lemmy.example.com/post/123
# ❌ Body content missing!
```

### After Our Change
```ruby
# Same Article
# Mastodon now displays:
# <h2>My Blog Post</h2>
# This is the full article text...
# https://lemmy.example.com/post/123
# ✅ Body content included!
```

## Running the Tests

To run just this test file:
```bash
bundle exec rspec spec/lib/activitypub/parser/status_parser_spec.rb
```

To run with detailed output:
```bash
bundle exec rspec spec/lib/activitypub/parser/status_parser_spec.rb --format documentation
```

## Test Output Example

```
ActivityPub::Parser::StatusParser
  correctly parses status
  when the likes collection is not inlined
    does not raise an error
  #quote_policy
    when nobody is allowed to quote
      returns a policy not allowing anyone to quote
    when everybody is allowed to quote
      returns a policy not allowing anyone to quote
    when everybody is allowed to quote but only followers are automatically approved
      returns a policy allowing everyone including followers
  #processed_text
    when object is a Note
      returns the content as-is
    when object is an Article
      includes title, summary, body content, and link
    when object is a Page
      includes title, body content, and link
    when object is an Event
      includes title, body content, and link
    when Article has no body content
      includes title and link without body content

Finished in 0.XX seconds
11 examples, 0 failures
```

## Key Points

1. **Test-Driven Development** - We wrote tests to verify our feature works
2. **Comprehensive Coverage** - Tests cover happy path and edge cases
3. **Backwards Compatible** - Tests ensure existing functionality still works
4. **Quality Assurance** - All tests must pass before merging

## Related Files

- **Implementation**: `app/lib/activitypub/parser/status_parser.rb`
- **Tests**: `spec/lib/activitypub/parser/status_parser_spec.rb` (this file)
- **Documentation**: `docs/screenshots/TEST_RESULTS.md`

## Summary

This test file is crucial for ensuring that ActivityPub content from federated platforms (Lemmy, Friendica, etc.) is correctly parsed and displayed in Mastodon. Our changes added tests to verify that body content is now included in the rendered output, fixing a longstanding issue where Article/Page/Event posts showed only headlines and links.
