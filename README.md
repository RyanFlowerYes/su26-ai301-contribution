# Contribution 2: Add Multi-Modal Support to Google Provider


**Contribution Number:** 2
**Student:** Ryan Ouardaoui 
**Issue:** https://github.com/truera/trulens/issues/2403
**Status:** Phase II  Complete

---

## Why I Chose This Issue

I chose this issue primarily because it is an enhancement issue in a very popular library. Additionally, it is completely empty and left unresolved for a few months, making it an issue where I have creative freedom in implementation, rather than debugging, and an issue with no controversy or competition. It is also a library in my skill level as it requires the implementation of multi-modal support in the Google Provider.
More precisely, I believe that this issue will help me grow as an AI engineer as it will make me understand LLMs deeper, through extensive research of Documentation and LLM evaluation itself in reading the other implementations of LLM providers in the library itself. Additionally, it will give me the skills needed to familiarize myself with multi-modal models: how they work, what they output, and how to use that output for evaluation; which I feel is a very important skill in today's ever evolving LLM landscape.

---

## Understanding the Issue

### Problem Description

The Gemini provider currently has no multimodal support. TruLens only handles text inputs, so files such as images, audio, video, or documents cannot be included in evaluations.

### Expected Behavior

The Google provider should accept media files with a valid MIME type and pass them to Gemini together with text input.

Existing text-only behavior should remain unchanged.

### Current Behavior

There is no supported file input format or provider endpoint for sending multimodal content to Gemini.


### Affected Components

This mainly affects the Google Gemini provider and its request-building logic.
---

## Reproduction Process

### Environment Setup

The environment needed to be the same as the one on the main branch. 

### Steps to Reproduce

Not applicable. This is a missing feature rather than a bug.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/RyanFlowerYes/trulens/tree/google-multimodal-support
- **Screenshots/logs:** [If applicable]
- **My findings:** The current provider only supports text input.

---

## Solution Approach

### Analysis

Gemini supports multimodal requests through structured content parts, but the current Google provider only accepts text.

The solution must add file handling while keeping existing text requests backward compatible.

### Proposed Solution

Extend the Google provider to accept structured inputs containing:

Text
File data
A valid MIME type

Convert these inputs into Gemini-compatible content parts and send them through the existing endpoint flow.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Add multimodal support to the Gemini provider without breaking text-only requests.

**Match:** Follow the existing Google provider, endpoint, and input-normalization patterns.

**Plan:** 
Add support for text and media content parts.
Validate file data and MIME types.
Convert inputs into Gemini-compatible request content.
Pass requests through the existing Google endpoint.
Add tests for valid and invalid multimodal inputs.
Update documentation with an example.
**Implement:** https://github.com/RyanFlowerYes/trulens/tree/google-multimodal-support

**Review:** 
Text-only requests still work

MIME types are validated

Errors are clear

Tests do not require live API credentials

No unrelated dependency changes

Linting and tests pass

**Evaluate:** Verify that text and media files are correctly passed to Gemini and that invalid inputs fail with clear errors.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
# su26-ai301-contribution
