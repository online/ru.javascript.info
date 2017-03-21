# Keyboard: keydown and keyup

Let's study keyboard events.

Before we start, please note that on modern devices there are other ways to "input something" then just a keyboard. For instance, people use speech recognition (tap microphone, say something, see it entered) or copy/paste with a mouse.

So if we want to track any input into an `<input>` field, then keyboard events is not enough. There's another event named `input` to handle changes of an `<input>` field, by any means. And it may be a better choice for such task. We'll cover it later in the chapter <info:events-change-input>.

Keyboard events should be used when we want to handle keyboard actions (virtual keyboard usually also counts). For instance, to react on arrow keys `key:Up` and `key:Down` or hotkeys (including combinations of keys).

[cut]


## Teststand [#keyboard-test-stand]

```offline
To better understand keyboard events, you can use the [teststand](sandbox:keyboard-dump).
```

```online
To better understand keyboard events, you can use the teststand below.

Try different key combinations in the text field.

[codetabs src="keyboard-dump" height=480]
```

As you read on, if you want to try things out -- return to the stand and press keys.


## Keydown and keyup

The `keydown` events happens when a key is pressed down, and then `keyup` -- when it's released.

### event.code and event.key

The `key` property of the event object allows to get the character, while the `code` property of the event object allows to get the "physical key code".

For instance, the same key `key:Z` can be pressed with or without `Shift`. That gives us two different characters: a lowercase and uppercase `z`, right?

The `event.key` is exactly the character, and it will be different. But `event.code` is the same:

| Key          | `event.key` | `event.code` |
|--------------|-------------|--------------|
| `key:Z`      |`z` (lowercase)         |`KeyZ`        |
| `key:Shift+Z`|`Z` (uppercase)          |`KeyZ`        |

```smart header="\"KeyZ\" and other key codes"
Key codes like `"KeyZ"` in the example above are described in the [UI Events code specification](https://www.w3.org/TR/uievents-code/).

For instance:
- Letter keys have codes `"Key<letter>"`: `"KeyA"`, `"KeyB"` etc.
- Digit keys have codes: `"Digit<number>"`: `"Digit0"`, `"Digit1"` etc.
- Special keys are coded by their names: `"Enter"`, `"Backspace"`, `"Tab"` etc.

See [alphanumeric section](https://www.w3.org/TR/uievents-code/#key-alphanumeric-section) for more examples, or just try the [teststand](#keyboard-test-stand) above.
```

```warn header="Case matters: `\"KeyZ\"`, not `\"keyZ\"`"
Seems obvious, but people still make mistakes.

Please evade mistypes: it's `KeyZ`, not `keyZ`. The check like `event.code=="keyZ"` won't work: the first letter of `"Key"` must be uppercase.
```

If a user works with different languages, then switching to another language would make a totally different character instead of `"Z"`. That will become the value of `event.key`, while `event.code` is always the same: `"KeyZ"`.

What is a key does not give any character? For instance, `key:Shift` or `key:Tab` or others. For those keys `event.key` is approximately the same as `event.code`:


| Key          | `event.key` | `event.code` |
|--------------|-------------|--------------|
| `key:Tab`      |`Tab`          |`Tab`        |
| `key:F1`      |`F1`          |`F1`        |
| `key:Backspace`      |`Backspace`          |`Backspace`        |
| `key:Shift`|`Shift`          |`ShiftRight` or `ShiftLeft`        |

Please note that `event.code` specifies exactly which key is pressed. For instance, most keyboards have two `key:Shift` keys: on the left and on the right side. The `event.code` tells us exactly which one was pressed, and `event.key` is responsible for the "meaning" of the key: what it is (a "Shift").

Let's say, we want to handle a hotkey: `key:Ctrl+Z` (or `key:Cmd+Z` for Mac). That's an "undo" action in most text editors. We can set a listener on `keydown` and check which key is pressed -- to detect when we have the hotkey.

How do you think, should we check the value of `event.key` or `event.code`?

Please, pause and answer.

Made up your mind?

If you've got an understanding, then the answer is, of course, `event.code`, and we don't want `event.key` here. The value of `event.key` can change depending on the language or `CapsLock` enabled. The value of `event.code` is strictly bound to the key, so here we go:

```js run
document.addEventListener('keydown', function(event) {
  if (event.code == 'KeyZ' && (event.ctrlKey || event.metaKey)) {
    alert('Undo!')
  }
});
```

## Auto-repeat

If a key is being pressed for a long enough time, it starts to repeat: the `keydown` triggers again and again, and then when it's released we finally get `keyup`. So it's kind of normal to have many `keydown` and a single `keyup`.

For all repeating keys the event object has `event.return=true`.


## Default actions

Default actions vary, as there are many possible things that may be initiated by keyboard.

For instance:

- A character appears on the screen (the most obvious outcome).
- A character is deleted (`key:Delete` key).
- The page is scrolled (`key:PageDown` key).
- The browser opens the "Save Page" dialog (`key:Ctrl+S`)
-  ...and so on.

Preventing the default action on `keydown` can cancel most of them, with the exception of OS-based special keys. For instance, on Windows `key:Alt+F4` closes the current browser window. And there's no way to stop it by preventing the default action in JavaScript.

For instance, the `<input>` below expects a phone number, so it does not accept keys except digits, `+`, `()` or `-`:

```html run
<script>
function checkPhoneKey(key) {
  return (key >= '0' && key <= '9') || key == '+' || key == '(' || key == ')' || key == '-';
}
</script>
<input *!*onkeydown="return checkPhoneKey(event.key)"*/!* placeholder="Phone, please" type="tel">
```

Please note that special keys like `key:Backspace`, `key:Left`, `key:Right`, `key:Ctrl+V` do not work. That's a side-effect effect of the strict filter `checkPhoneKey`.

Let's relax it a little bit:


```html run
<script>
function checkPhoneKey(key) {
  return (key >= '0' && key <= '9') || key == '+' || key == '(' || key == ')' || key == '-' ||
    key == 'ArrowLeft' || key == 'ArrowRight' || key == 'Delete' || key == 'Backspace';
}
</script>
<input *!*onkeydown="return checkPhoneKey(event.key)"*/!* placeholder="Phone, please" type="tel">
```

Now arrows and deletion works well.

But we still can enter anything by using a mouse and right-click + Paste. So the filter is not 100% reliable. We can just let it be like that, because that usually works. Or an alternative approach would be to track the `input` event -- it triggers after any modification. There we can check the new value and highlight/modify it when it's invalid.

## Legacy

In the past, there was a `keypress` event, and also `keyCode`, `charCode`, `which` properties of the event object.

There were so many browser incompatibilities that standard developers decided to deprecate all of them and remove from the specification. The old code still works, as the browser keep supporting them, but there's totally no need to use those any more.

There was time when this chapter included their detailed description. But as of now we can forget about those.


## Summary

Pressing a key always generates a keyboard event, be it symbol keys or special keys like `key:Shift` or `key:Ctrl` and so on. The only exception is `key:Fn` key that is sometimes present on laptop keyboards. There's no keyboard event for it, because it's often implemented on lower level than OS.

Keyboard events:

- `keydown` -- on press the key (auto-repeats if the key is pressed for long),
- `keyup` -- on releasing the key.

Main keyboard event properties:

- `code` -- the "key code" (`"KeyA"`, `"ArrowLeft"` and so on), specific to the physical location of the key on keyboard.
- `key` -- the character (`"A"`, `"a"` and so on), for non-character keys usually has the same value  as `code`.

In the past, keyboard events were sometimes used to track user input in form fields. That's not the case any more, because we have `input` and `change` events for that (to be covered later). They are better, because they work for all ways of input, including mouse or speech recognition.

We should use keyboard events when we really want keyboard. For example, to react on hotkeys or special keys.