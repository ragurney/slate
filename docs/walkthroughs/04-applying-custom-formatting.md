# Applying Custom Formatting

In the previous guide we learned how to create custom block types that render chunks of text inside different containers. But Slate allows for more than just "blocks".

In this guide, we'll show you how to add custom formatting options, like **bold**, _italic_, `code` or ~~strikethrough~~.

So we start with our app from earlier:

```js
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const renderElement = useCallback(props => {
    switch (props.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  return (
    <Slate editor={editor} defaultValue={defaultValue}>
      <Editable
        renderElement={renderElement}
        onKeyDown={event => {
          if (event.key === '`' && event.ctrlKey) {
            event.preventDefault()
            const { selection } = editor
            const [node] = Editor.nodes(editor, { match: { type: 'code' } })
            const isCodeActive = !!node
            Editor.setNodes(
              editor,
              { type: isCodeActive ? 'paragraph' : 'code' },
              { match: 'block' }
            )
          }
        }}
      />
    </Slate>
  )
}
```

And now, we'll edit the `onKeyDown` handler to make it so that when you press `control-B`, it will add a `bold` format to the currently selected text:

```js
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const renderElement = useCallback(props => {
    switch (prop.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  return (
    <Slate editor={editor} defaultValue={defaultValue}>
      <Editable
        renderElement={renderElement}
        onKeyDown={event => {
          if (!event.ctrlKey) {
            return
          }

          switch (event.key) {
            // When "`" is pressed, keep our existing code block logic.
            case '`': {
              event.preventDefault()
              const [node] = Editor.nodes(editor, { match: { type: 'code' } })
              const isCodeActive = !!node
              Editor.setNodes(
                editor,
                { type: isCodeActive ? null : 'code' },
                { match: 'block' }
              )
              break
            }

            // When "B" is pressed, bold the text in the selection.
            case 'b': {
              event.preventDefault()
              Editor.setNodes(
                editor,
                { bold: true },
                // Apply it to text nodes, and split the text node up if the
                // selection is overlapping only part of it.
                { match: 'text', split: true }
              )
              break
            }
          }
        }}
      />
    </Slate>
  )
}
```

Okay, so we've got the hotkey handler setup... but! If you happen to now try selecting text and hitting `Ctrl-B`, you won't notice any change. That's because we haven't told Slate how to render a "bold" mark.

For every format you want to add to your schema, Slate will break up the text content into "leaves", and you need to tell Slate how to read it, just like for elements. So let's define a `Leaf` component:

```js
// Define a React component to render leaves with bold text.
const Leaf = props => {
  return (
    <span
      {...props.attributes}
      style={{ fontWeight: leaf.bold ? 'bold' : 'normal' }}
    >
      {props.children}
    </span>
  )
}
```

Pretty familiar, right?

And now, let's tell Slate about that leaf. To do that, we'll pass in the `renderLeaf` prop to our editor. Also, let's allow our formatting to be toggled by adding active-checking logic.

```js
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const renderElement = useCallback(props => {
    switch (props.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  // Define a leaf rendering function that is memoized with `useCallback`.
  const renderLeaf = useCallback(props => {
    return <Leaf {...props} />
  }, [])

  return (
    <Slate editor={editor} defaultValue={defaultValue}>
      <Editable
        renderElement={renderElement}
        // Pass in the `renderLeaf` function.
        renderLeaf={renderLeaf}
        onKeyDown={event => {
          if (!event.ctrlKey) {
            return
          }

          switch (event.key) {
            case '`': {
              event.preventDefault()
              const [node] = Editor.nodes(editor, { match: { type: 'code' } })
              const isCodeActive = !!node
              Editor.setNodes(
                editor,
                { type: isCodeActive ? null : 'code' },
                { match: 'block' }
              )
              break
            }

            case 'b': {
              event.preventDefault()
              Editor.setNodes(
                editor,
                { bold: true },
                { match: 'text', split: true }
              )
              break
            }
          }
        }}
      />
    </Slate>
  )
}

const Leaf = props => {
  return (
    <span
      {...props.attributes}
      style={{ fontWeight: leaf.bold ? 'bold' : 'normal' }}
    >
      {props.children}
    </span>
  )
}
```

Now, if you try selecting a piece of text and hitting `Ctrl-B` you should see it turn bold! Magic!
