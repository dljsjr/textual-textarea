# Textual Textarea
![Textual Textarea Screenshot](textarea.png)

## Installation

```
pip install textual-textarea
```

## Features
Full-featured text editor experience with VS-Code-like bindings, in your Textual App:
- Syntax highlighting and support for themes.
- Move cursor and scroll with mouse or keys (including <kbd>ctrl+arrow</kbd>, <kbd>PgUp/Dn</kbd>,  <kbd>Home/End</kbd>).
- Select text using <kbd>shift</kbd> or click and drag.
- Open (<kbd>ctrl+o</kbd>) and save (<kbd>ctrl+s</kbd>) files.
- Cut (<kbd>ctrl+x</kbd>), copy (<kbd>ctrl+c</kbd>), paste (<kbd>ctrl+u/v</kbd>), optionally using the system clipboard.
- Comment selections with <kbd>ctrl+/</kbd>.
- Indent and dedent (optionally for a multiline selection) to tab stops with <kbd>Tab</kbd> and <kbd>shift+Tab</kbd>.
- Automatic completions of quotes and brackets.
- Quit with <kbd>ctrl+q</kbd>.

## Usage

### Initializing the Widget

The TextArea is a Textual Widget. You can add it to a Textual
app using `compose` or `mount`:

```python
from textual_textarea import TextArea
from textual.app import App, ComposeResult

class TextApp(App, inherit_bindings=False):
    def compose(self) -> ComposeResult:
        yield TextArea(language="python", theme="solarized-dark")

    def on_mount(self) -> None:
        ta = self.query_one(TextArea)
        ta.focus()

app = TextApp()
app.run()
```

In addition to the standard Widget arguments, TextArea accepts three additional, optional arguments when initializing the widget:

- language (str): Must be `None` or the short name of a [Pygments lexer](https://pygments.org/docs/lexers/), e.g., `python`, `sql`, `as3`. Defaults to `None`.
- theme (str): Must be name of a [Pygments style](https://pygments.org/styles/), e.g., `bw`, `github-dark`, `solarized-light`. Defaults to `monokai`.
- use_system_clipboard (bool): Set to `False` to make the TextArea's copy and paste operations ignore the system clipboard. Defaults to `True`. Some Linux users may need to apt-install `xclip` or `xsel` to enable the system clipboard features.

The TextArea supports many actions and key bindings. **For proper binding of `ctrl+c` to the COPY action,
you must initialize your App with `inherit_bindings=False`** (as shown above), so that `ctrl+c` does not quit the app. The TextArea implements `ctrl+q` as quit; you way wish to mimic that in your app so that other in-focus widgets use the same behavior.

### Interacting with the Widget

#### Getting and Setting Text

The TextArea exposes a `text` property that contains the full text contained in the widget. You can retrieve or set the text by interacting with this property:

```python
ta = self.query_one(TextArea)
old_text = ta.text
ta.text = "New Text!\n\nMany Lines!"
```


#### Getting and Setting The Cursor Position

The TextArea exposes a `cursor` property that returns a NamedTuple with the position of the cursor. The tuple is (line_number, x_pos):

```python
ta = self.query_one(TextArea)
old_cursor = ta.cursor
ta.cursor = (999, 0)  # the cursor will move as close to line 999, pos 0 as possible
cursor_line_number = ta.cursor.lno
cursor_x_position = ta.cursor.pos
```

#### Getting and Setting The Language

Syntax highlighting and comment insertion depends on the configured language for the TextArea.

The TextArea exposes a `language` property that returns `None` or a string that is equal to the short name of the [Pygments lexer](https://pygments.org/docs/lexers/) for the currently configured language:

```python
ta = self.query_one(TextArea)
old_language = ta.language
ta.language = "python"
```

#### Getting Theme Colors

If you would like the rest of your app to match the colors from the TextArea's theme, they are exposed via the `theme_colors` property.

```python
ta = self.query_one(TextArea)
color = ta.theme_colors.contrast_text_color
bgcolor = ta.theme_colors.bgcolor
highlight = ta.theme_colors.selection_bgcolor
```


#### Adding Bindings and other Behavior

You can subclass TextArea to add your own behavior. This snippet adds an action that posts a Submitted message containing the text of the TextArea when the user presses <kbd>ctrl+j</kbd>:

```python
from textual.message import Message
from textual_textarea import TextArea


class CodeEditor(TextArea):
    BINDINGS = [
        ("ctrl+j", "submit", "Run Query"),
    ]

    class Submitted(Message, bubble=True):
        def __init__(self, text: str) -> None:
            super().__init__()
            self.text = text

    async def action_submit(self) -> None:
        self.post_message(self.Submitted(self.text))
```
