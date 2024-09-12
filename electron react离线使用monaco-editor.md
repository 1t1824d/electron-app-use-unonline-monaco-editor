# electron react离线使用monaco-editor

### 1.安装@monaco-editor/react和monaco-editor

```
pnpm i @monaco-editor/react

```

```
pnpm i monaco-editor
```

### 2.引入并做monaco-editor离线配置

```
import Editor, { DiffEditor, useMonaco, loader } from '@monaco-editor/react'
```

```
import * as monaco from 'monaco-editor'
import editorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker';
import jsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker';
import cssWorker from 'monaco-editor/esm/vs/language/css/css.worker?worker';
import htmlWorker from 'monaco-editor/esm/vs/language/html/html.worker?worker';
import tsWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker';

```
```
self.MonacoEnvironment = {
  getWorker(_, label) {
    if (label === 'json') {
      return new jsonWorker();
    }
    if (label === 'css' || label === 'scss' || label === 'less') {
      return new cssWorker();
    }
    if (label === 'html' || label === 'handlebars' || label === 'razor') {
      return new htmlWorker();
    }
    if (label === 'typescript' || label === 'javascript') {
      return new tsWorker();
    }
    return new editorWorker();
  },
};
loader.config({ monaco })
loader.init().then(/* ... */);
```

### react中使用

```
function App(): JSX.Element {
  const editorRef = useRef(null)
  function handleEditorDidMount(editor, monaco) {
    editorRef.current = editor
  }
  function showValue() {
    alert(editorRef.current.getValue())
  }
  return (
    <>
      <Editor
        width="600px"
        height="30vh"
        defaultLanguage="javascript"
        defaultValue="// some comment"
        onMount={handleEditorDidMount}
      />

      <div className="actions">
        <div className="action">
          <a target="_blank" rel="noreferrer" onClick={showValue}>
          Show value
          </a>
        </div>
      </div>
    </>
  )
}

export default App
```

### 完整代码示例

```
import Versions from './components/Versions'
import electronLogo from './assets/electron.svg'
import { useRef } from 'react'
/////////////////////
import Editor, { DiffEditor, useMonaco, loader } from '@monaco-editor/react'
import * as monaco from 'monaco-editor'
import editorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker';
import jsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker';
import cssWorker from 'monaco-editor/esm/vs/language/css/css.worker?worker';
import htmlWorker from 'monaco-editor/esm/vs/language/html/html.worker?worker';
import tsWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker';
self.MonacoEnvironment = {
  getWorker(_, label) {
    if (label === 'json') {
      return new jsonWorker();
    }
    if (label === 'css' || label === 'scss' || label === 'less') {
      return new cssWorker();
    }
    if (label === 'html' || label === 'handlebars' || label === 'razor') {
      return new htmlWorker();
    }
    if (label === 'typescript' || label === 'javascript') {
      return new tsWorker();
    }
    return new editorWorker();
  },
};
loader.config({ monaco })
loader.init().then(/* ... */);
/////////////////////
function App(): JSX.Element {
  const ipcHandle = (): void => window.electron.ipcRenderer.send('ping')
  const editorRef = useRef(null)
  function handleEditorDidMount(editor, monaco) {
    editorRef.current = editor
  }
  function showValue() {
    alert(editorRef.current.getValue())
  }
  return (
    <>
      <img alt="logo" className="logo" src={electronLogo} />
      <Editor
        width="600px"
        height="30vh"
        defaultLanguage="javascript"
        defaultValue="// some comment"
        onMount={handleEditorDidMount}
      />

      <div className="actions">
        <div className="action">
          <a target="_blank" rel="noreferrer" onClick={showValue}>
          Show value
          </a>
        </div>
        <div className="action">
          <a target="_blank" rel="noreferrer" onClick={ipcHandle}>
            Send IPC
          </a>
        </div>
      </div>
      <Versions></Versions>
    </>
  )
}

export default App

```
## monaco-editor离线配置官方说明

### loader-config

该库导出（命名）名为loader实用程序。基本上，它是@monaco-editor/loader的引用。默认情况下， monaco文件是从CDN下载的。有能力改变这种行为，以及有关monaco AMD加载程序的其他事情。我们有一个默认的配置文件，您可以通过以下方式修改：
```
import { loader } from '@monaco-editor/react';

// you can change the source of the monaco files
loader.config({ paths: { vs: '...' } });

// you can configure the locales
loader.config({ 'vs/nls': { availableLanguages: { '*': 'de' } } });

// or
loader.config({
  paths: {
    vs: '...',
  },
  'vs/nls': {
    availableLanguages: {
      '*': 'de',
    },
  },
});
```
### 使用monaco-editor作为 npm 包
从v4.4.0版本开始，可以将monaco-editor作为npm包使用；从node_modules导入它并将monaco源包含到您的包中（而不是使用 CDN）。要使其正常工作，您可以执行以下操作：

```
import * as monaco from 'monaco-editor';
import { loader } from '@monaco-editor/react';

loader.config({ monaco });

// ...
```

注意：您应该意识到，这可能需要额外的webpack插件，例如monaco-editor-webpack-plugin ，否则可能无法在CRA生成的应用程序中使用而不弹出它们。

如果你使用Vite ，你需要这样做：

```
import { loader } from '@monaco-editor/react';

import * as monaco from 'monaco-editor';
import editorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker';
import jsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker';
import cssWorker from 'monaco-editor/esm/vs/language/css/css.worker?worker';
import htmlWorker from 'monaco-editor/esm/vs/language/html/html.worker?worker';
import tsWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker';

self.MonacoEnvironment = {
  getWorker(_, label) {
    if (label === 'json') {
      return new jsonWorker();
    }
    if (label === 'css' || label === 'scss' || label === 'less') {
      return new cssWorker();
    }
    if (label === 'html' || label === 'handlebars' || label === 'razor') {
      return new htmlWorker();
    }
    if (label === 'typescript' || label === 'javascript') {
      return new tsWorker();
    }
    return new editorWorker();
  },
};

loader.config({ monaco });

loader.init().then(/* ... */);
```
注意：您传递的对象将与默认对象深度合并


参考链接：
1.https://www.npmjs.com/package/@monaco-editor/react#use-monaco-editor-as-an-npm-package

2.https://www.npmjs.com/package/monaco-editor

3.https://microsoft.github.io/monaco-editor/
