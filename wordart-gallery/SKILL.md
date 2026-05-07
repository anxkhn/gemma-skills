---
name: wordart-gallery
description: Create an animated gallery of WordArt styles for a supplied name using @codeweaver/wordart.
metadata:
  homepage: https://github.com/anxkhn/gemma-skills/tree/main/wordart-gallery
---

# WordArt Gallery

## Instructions

Use this skill when the user asks for WordArt, animated name art, stylized text,
or a gallery of visual text treatments.

Call the `run_js` tool with the following exact parameters:

- script name: index.html
- data: A JSON string with the following fields:
  - name: String. The name or text to display. If the user says "my name" and
    does not provide another name, use "Anas Khan".
  - size: Number, optional. Font size scale. Default: 3.
  - speed: Number, optional. Animation speed. Default: 2.

When the tool returns, show the webview to the user. Keep the chat response
short because the visual gallery is the main output.
