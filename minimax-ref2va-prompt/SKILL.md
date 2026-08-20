---
name: minimax-ref2va-prompt
description: Convert reference image tags and a final scene description into a strict 6-block English Context-IR prompt for the MiniMax H3-Base Ref2VA full-reference video model (local H3-Base or ComfyUI). Use when the user asks to build a MiniMax Ref2VA video-edit prompt from character tags plus a scene description.
---

# MiniMax Ref2VA Prompt (H3-Base Ref2VA Full-Reference Mode)

Generate a video-editing prompt that replaces the original subject in the source video with a character defined by the user's reference images. The output must follow the official Context-IR 6-block format exactly and be designed for local H3-Base or ComfyUI execution.

## Input

1. **Reference Image Tags** - tags describing the character, for 1 or 2 images.
2. **Final Scene Description** - the user's description of the desired environment, actions, and camera movement, written in their target visual style.

## Reference

The authoritative full-reference mode output format guide is in [references/ref-en.txt](references/ref-en.txt). Read it when the user asks for output based on the official guide, or when clarification is needed on reference label semantics (`<Subject N>`, `<Picture N>`, `<Video N>`, `<Audio N>`), relationship markers, section rules, or the complete worked example. The template below distills the specific video-editing workflow; the guide is the authoritative source for format details.

## Hard Constraints (generation fails if violated)

- **Pure English, single version**: output only English, in ONE code block. No bilingual versions.
- **Mandatory 6-block structure**, in this exact order: `subject_definitions`, `summary`, `retention_analysis`, `detailed_description`, `overall_soundscape`, `non_diegetic_music`.
- **Indexing starts at 1**: use `<Video 1>` and `<Picture 1>`; include `<Picture 2>` only if the user provided a second image.
- **Subject mapping**: if two images are provided, combine them to define one subject: `<Subject 1> is the character whose appearance is derived from <Picture 1> and <Picture 2>...`
- Never describe the subject from the original video; always describe `<Subject 1>` using the reference picture tags.

### Per-block rules

- `subject_definitions`: define `<Video 1> is the source video for the target video edit.` Do NOT create standalone `<Picture>` definitions; define `<Subject 1>` directly using the picture references.
- `summary`: must begin with exactly `[video editing + reference generation] The target video is an edited version of <Video 1>.`
- `retention_analysis`:
  - `<Video 1>` MUST be `partially_preserved - the background environment and camera movement are retained, while the original subject is completely replaced.`
  - `<Subject 1>` MUST be `fully_preserved`.
- `detailed_description`:
  - The first sentence declares the target visual style, extracted from the user's scene description.
  - Begin the scene with `[Shot 1]` - absolutely NO timestamp on the first shot.
  - **Action & trait binding**: `<Subject 1>` MUST explicitly execute the action. Force 1-2 specific physical traits in motion (e.g., "her hair sways").
  - Integrate camera movement as "Type + Amplitude + Speed".
- `overall_soundscape`: one sentence of basic physical ambient sound based on the scene, or `N/A`.
- `non_diegetic_music`: `N/A`.

## Interaction Rules

1. **Incomplete input**: if the Reference Image Tags or Final Scene Description is missing, incomplete, or ambiguous, ask the user for the missing information first; do not invent or auto-complete inputs.
2. **Tag vs. scene conflicts**: if the Reference Image Tags conflict with the Final Scene Description (e.g., hair color or clothing), the character's appearance follows the tags; the scene description governs environment, action, and camera movement.
3. **Pure audio / no-replacement tasks**: when the task has no subject replacement (e.g., pure audio work), state `no replacement` in `subject_definitions`, omit subject-replacement logic from `summary` and `retention_analysis`, and expand `overall_soundscape` and `non_diegetic_music` in detail as the task requires. For audio label and task-type details, see [references/ref-en.txt](references/ref-en.txt).

## Output Template

Follow this template exactly:

```text
subject_definitions:
<Video 1> is the source video for the target video edit.
<Subject 1> is the character from <Picture 1> [and <Picture 2> if a second image is provided], described as [fluent English description based on the provided Tags].

summary:
[video editing + reference generation] The target video is an edited version of <Video 1>. The original subject is replaced by <Subject 1>, who performs the actions seamlessly within the newly defined environment style.

retention_analysis:
<Video 1> (background and camera motion): partially_preserved - the environment structure and camera movement of the source video are retained, while the original subject is replaced.
<Subject 1> (appears in [Shot 1]): fully_preserved - the appearance, clothing, and character traits from the picture(s) are strictly retained.

detailed_description:
The target video is in a [extract visual style from user's Final Scene Description] style.
[Shot 1] [Establish the scene environment in one sentence]. <Subject 1>, the [1-2 core identifiers, e.g., young girl with blue hair], [executes the action]. [Force the description of 1-2 physical traits in motion]. [Naturally integrate the camera movement].

overall_soundscape:
[Write 1 sentence of basic physical ambient sound based on the scene, or N/A]

non_diegetic_music:
N/A
```

## Workflow

1. Take the user's Reference Image Tags and Final Scene Description as inputs.
2. Build the `<Subject 1>` description from the tags (combine both pictures when two are provided).
3. Fill every block following the template and hard constraints above.
4. Return the complete prompt in a single English code block.
