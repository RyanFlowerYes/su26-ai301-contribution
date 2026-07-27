# Contribution 2: Add Multi-Modal Support to Google Provider


**Contribution Number:** 2
**Student:** Ryan Ouardaoui 
**Issue:** https://github.com/truera/trulens/issues/2403
**Status:** Phase IV  complete

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

All tests live in `tests/unit/test_google_multimodal.py` and run without live Google credentials. The Gemini SDK constructors (`types.Part.from_text`, `types.Part.from_bytes`, `types.Content`) and `GenerateContentConfig` are mocked with `monkeypatch`, so the tests verify *my* translation logic rather than google-genai's internals. The module is guarded with `pytest.importorskip` for both `trulens.providers.google` and `google.genai`, and the classes are marked `@pytest.mark.optional`.

### Unit Tests

**`TestGoogleMultimodalParts`** — the `_to_gemini_part` converter:
- [x] `test_converts_string_to_text_part`: a bare `str` becomes a text part via `from_text` (and `from_bytes` is never called).
- [x] `test_converts_text_dictionary_to_text_part`: `{"type": "text", "text": ...}` becomes a text part.
- [x] `test_converts_media_dictionary_to_bytes_part`: `{"type": "media", "data": <bytes>, "mime_type": ...}` becomes a bytes part via `from_bytes`.
- [x] `test_media_requires_explicit_mime_type`: a media part with no `mime_type` raises `ValueError` ("require an explicit 'mime_type'").
- [x] `test_rejects_unsupported_content_parts`: parametrized over `123`, `None`, `["text"]`, `{"type": "video"}`, `{"type": "audio", "data": ...}` — each raises `ValueError` ("Unsupported content part").

**`TestGoogleMultimodalCompletion`** — end-to-end request building in `_create_chat_completion`:
- [x] `test_builds_mixed_text_and_image_message`: a single user message mixing text + image produces the expected `contents` passed to `generate_content`.
- [x] `test_preserves_multimodal_part_order`: text → media → text ordering is preserved in the outgoing parts.
- [x] `test_supports_multiple_user_messages`: multiple user messages (string content + media-list content) each become their own `Content` entry.
- [x] `test_adds_system_instruction_to_config`: a `system` message is routed into the request config, not the user parts.
- [x] `test_uses_custom_seed`: a caller-supplied seed flows through to the config.
- [x] `test_prompt_path_still_works`: the legacy `prompt=` path (no `messages`) still builds a valid text request — backward compatibility.
- [x] `test_requires_prompt_or_messages`: calling with neither `prompt` nor `messages` raises `ValueError`.

**`TestGoogleMultimodalStructuredOutput`**:
- [x] `test_returns_parsed_structured_response`: when Gemini returns a parsed structured object, the provider surfaces it correctly.

### Integration Tests

- Not included by design. The tests deliberately avoid live Gemini calls (no credentials required in CI). Real-network verification was done manually — see below.

### Manual Testing

Manual testing consisted of trying to see if everything ran as intended and that the new implementation did not break previous logic.

---

## Implementation Notes

## Implementation Notes

### Phase III Progress (implementation — commits dated Jul 11, 2026)

**What I built:**
- Added `from google.genai import types` to `provider.py`.
- Added a new helper, `_to_gemini_part(self, part)`, that normalizes a content part into a Gemini `types.Part`:
  - `str` → `types.Part.from_text(text=...)`
  - `{"type": "text", "text": ...}` → text part
  - `{"type": "media", "data": <bytes>, "mime_type": ...}` → `types.Part.from_bytes(...)`
  - missing `mime_type` on a media part raises a clear `ValueError`
  - anything else raises `ValueError` with a message showing the accepted shapes
- Rewrote the user-message branch of `_create_chat_completion`: string content still maps to a single text part, while list content is mapped through `_to_gemini_part`, and the result is wrapped in a typed `types.Content(role="user", parts=parts)` instead of the old raw dict.
- Migrated the legacy `prompt=` path to the same typed `types.Content` / `types.Part.from_text` construction so both code paths are consistent.
- Added `tests/unit/test_google_multimodal.py` with 15 tests across 3 classes (see Testing Strategy).

**Challenges faced / decisions made:**
- The original provider built request content as plain dicts (`{"parts": [{"text": ...}]}`). Supporting media meant switching to the SDK's typed `types.Content` / `types.Part` objects. I kept the string-content fast path so existing text-only evaluations behave identically (verified by `test_prompt_path_still_works`).
- Biggest effort was making the tests credential-free and hermetic: I mocked `types.Part.from_text`, `types.Part.from_bytes`, and `types.Content` with `monkeypatch` so the assertions check *my* translation logic, not google-genai's internal Pydantic models. Added `pytest.importorskip` guards so the file is skipped cleanly when the optional deps aren't installed.
- Chose to require an explicit `mime_type` on media parts (rather than guessing) and to fail fast with descriptive `ValueError`s on unsupported shapes, so misuse is caught locally instead of surfacing as an opaque SDK error.
- Post-implementation cleanup: ran `ruff` formatting and removed unrelated dependency-file changes that had been picked up, keeping the diff scoped to the two intended files.

**Commits this week:**
- `831ec0a`: Add multi-modal support to TruLens
- `1715fb0`: Merge branch 'truera:main' into google-multimodal-support
- `4471f30`: ruff-format fixes
- `6bcb718`: Remove unrelated dependency file changes
- `4daf1d8`: ruff formatting + missing importorskip

### Code Changes

- **Files modified:**
  - `src/providers/google/trulens/providers/google/provider.py` (+44 / −9)
  - `tests/unit/test_google_multimodal.py` (new file, +425)
- **Key commits:**
  - `831ec0a` — core feature (`_to_gemini_part` + `_create_chat_completion` rewrite + tests): https://github.com/RyanFlowerYes/trulens/commit/831ec0a
  - `6bcb718` — scope cleanup (drop unrelated dependency changes): https://github.com/RyanFlowerYes/trulens/commit/6bcb718
  - `4daf1d8` — formatting + missing `importorskip`: https://github.com/RyanFlowerYes/trulens/commit/4daf1d8
- **Approach decisions:**
  - Typed `types.Content` / `types.Part` over raw dicts, for correctness and forward-compatibility with the Gemini SDK.
  - Backward compatibility preserved for both the `messages=` string path and the legacy `prompt=` path.
  - Explicit `mime_type` requirement + descriptive `ValueError`s for invalid inputs.
  - Credential-free, mock-based unit tests marked optional and guarded by `importorskip`.

---
## Pull Request

**PR Link:** (https://github.com/truera/trulens/pull/2596)

**PR Description:** # Description

Adds multi-modal (text + image/audio/video) input support to the Google (Gemini)
feedback provider, completing the existing `TODO` in
`src/providers/google/trulens/providers/google/provider.py`.

Previously, `Google._create_chat_completion` assumed `message["content"]` was
always a plain string and wrapped it in a single text `Part`. There was no way to
pass media through the provider, so users wanting multi-modal evaluations had to
subclass `Google` and call `google_client.models.generate_content` directly,  as
shown in `examples/expositional/models/google/gemini_multi_modal_evaluation.ipynb`.
That workaround bypasses `self.endpoint` entirely, meaning those calls receive no
cost tracking, token counting, or rate-limit pacing.

`content` may now be either:

- a `str` (unchanged behavior), or
- a list of content parts:
  - `"some text"`
  - `{"type": "text", "text": str}`
  - `{"type": "media", "data": bytes, "mime_type": str}`

Media parts are converted to `types.Part.from_bytes` and cover images, audio, and
video, any MIME type Gemini accepts. Example:

```python
provider = Google()
result = provider._create_chat_completion(
    messages=[{"role": "user", "content": [
        {"type": "text", "text": "Does this restaurant have outdoor seating?"},
        {"type": "media", "data": image_bytes, "mime_type": "image/png"},
    ]}],
    response_format=ImageFaithfulnessScore,
)
```

Also added a few tests to validate my implementation in 'tests/unit/test_google_multimodal.py'. Happy to change it up based on maintainer feedback!

## Other details good to know for developers

- **Existing text evaluations are unaffected.** String content takes the same path
  as before and produces an identical payload. This is explicitly covered by tests.
- **Design decision, explicit `mime_type`.** Media parts require `mime_type`
  rather than defaulting or inferring it. Gemini cannot decode inline bytes without
  it, and a wrong default (e.g. assuming `image/png` for a WAV) fails in confusing
  ways. Missing `mime_type` raises a `ValueError`.
- **Design decision, one `media` type, not `image`.** The example notebook
  demonstrates image, audio, and video evaluations that differ only in MIME type.
  A single `media` part covers all three and extends to future modalities without a
  schema change.
- To my knowledge, there was no pre-existing multi-modal message convention anywhere in TruLens core
  (`llm_provider.py` only ever emits `content: str`), so this schema is new. Happy
  to align it with a different shape if maintainers prefer one.
- `types.Content` and `types.Part` are now used consistently for both the `messages`
  and `prompt` paths, replacing the hand-built dicts.

**Follow-up (not in this PR):** inline media is limited to ~20MB per request. Larger
files require the Gemini Files API (`client.files.upload` + `Part.from_uri`), which
needs caching to avoid re-uploading per evaluation call and does not exist on Vertex
AI clients. Happy to take that on in a separate PR.

## Type of change

- [ ] Bug fix (non-breaking change which fixes an issue)
- [x] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to
  not work as expected)
- [x] New Tests
- [ ] This change includes re-generated golden test results
- [ ] This change requires a documentation update


fixes #2403 

**Maintainer Feedback:**
- Feedback was fully positive

**Status:** Merged.

---

## Learnings & Reflections

### Technical Skills Gained

Through contributing to TruLens, I gained hands-on experience working with the Gemini SDK and integrating multimodal capabilities into an existing production codebase. A large part of the learning process involved reading and understanding the Gemini API documentation, experimenting with different models, and debugging issues related to file uploads, model availability, and response handling. I also became more comfortable navigating a mature open-source project, following contribution guidelines, and writing code that fit the project's existing architecture and testing standards.

### Challenges Overcome

The biggest challenge was working through unfamiliar documentation while ensuring my implementation followed the project's contribution standards. The Gemini SDK was evolving, so some examples were outdated or behaved differently than expected, which required a lot of experimentation and careful reading of the documentation. I also had to adhere to the repository's coding conventions and review feedback, making sure my implementation was both technically correct and maintainable. Breaking the work into smaller steps, testing frequently, and iterating on feedback helped me successfully complete the feature.

### What I'd Do Differently Next Time

Overall, I'm happy with how the project turned out and with the quality of my contribution. If I were to do it again, I would aim to move more quickly from understanding the problem to opening a pull request. I sometimes spent extra time double-checking implementation details before sharing my work. Becoming more comfortable proposing an initial solution earlier and refining it through code review would likely make my contributions both faster and more efficient.
---

## Resources Used

- (https://googleapis.github.io/python-genai/)
# su26-ai301-contribution
