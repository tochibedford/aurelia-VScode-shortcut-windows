# aurelia-shortcut-windows

I use this method to quickly create new aurelia components while I work in VSCode. It also binds a hotkey `Ctrl + Shift + Alt + 1` for you to activate it.
You're able to use this hotkey when you select a folder in the VSCode File explorer that you want to component to be created inside of. When the hotkey is activated it will ask for the name of the component, preferrably use spinal-case like in the Aurelia Docs. It will then create a folder with the component-name, and three files in the created folder (component-name.ts, component-name.html, component-name.scss).

- First Install "multi-command" and "macro-commander (Command Runner)" in VSCode
![image](https://user-images.githubusercontent.com/34871260/220627995-e09234b9-22ad-4506-9a2b-9a5db9ddab1b.png)

- Then open up the `settings.json` file for VSCode. This can be done by hitting `Ctrl + Shift + P` then searching for and clicking on `Preferences: Open User Settings (JSON)` in the search box that comes up
![image](https://user-images.githubusercontent.com/34871260/220628862-3b3114b9-8c3a-4336-bc84-02c548e30b07.png)

- In the `settings.json` file add the following entry and save the file.
```
"macros": {
    "createAureliaComponentFolderAndFiles" : [
      { 
        "javascript": [
  
          "const newAureliaFolder = await window.showInputBox()",  // get the component's name
  
              // create a object to store various edits, including file creation, deletion and edits
          "const we = new vscode.WorkspaceEdit();",
  
          "const clip = await vscode.env.clipboard.readText()",
              // get the workspace folder path + file:/// uri scheme
          "const thisWorkspace = vscode.workspace.workspaceFolders[0].uri.toString();",  // file:///c:/Users/Tochi/OneDrive/TochiBedford
              // append the new folder name
          "const uriBase = await `${thisWorkspace}/${encodeURIComponent(clip).replaceAll('%5C', '/')}/${newAureliaFolder}`;",  //file:///c:/Users/Tochi/OneDrive/TochiBedford/Test
  
          // "await window.showInformationMessage(` ${thisWorkspace}`)",  // just handy to check on progress
  
              // create uri's for the three files to be created
          "let newUri1 = vscode.Uri.parse(`${uriBase}/${newAureliaFolder}.ts`);",
          "let newUri2 = vscode.Uri.parse(`${uriBase}/${newAureliaFolder}.scss`);",
          "let newUri3 = vscode.Uri.parse(`${uriBase}/${newAureliaFolder}.html`);",
  
          "let newFiles = [ newUri1, newUri2, newUri3];",
  
          "for (const newFile of newFiles) { let document = await we.createFile(newFile, { ignoreIfExists: false, overwrite: true });};",
  
          "let htmlContent = `<template></template>`",
          "let viewModelName = newAureliaFolder.split('-').map((item, index)=>{",
                                  "if(index === 0){ return(item) }",
                              "return(item.slice(0,1).toUpperCase() + item.slice(1,)) }).join('')",
                //  insert text into the top of each file
          "await we.insert(newUri1, new vscode.Position(0, 0), `export class ${viewModelName} {\n}`);",
          // "await we.insert(newUri2, new vscode.Position(0, 0), ``);",
          "await we.insert(newUri3, new vscode.Position(0, 0), htmlContent);",
          
          "await vscode.workspace.applyEdit(we);",  // apply all the edits: file creation and adding text
          
          // "await window.showInformationMessage(uriBase)",
          "for (const newFile of newFiles) { let document = await vscode.workspace.openTextDocument(newFile); await document.save();};",
        ]
      }
    ]
  },
```
adapted from: https://stackoverflow.com/a/64289429/9094171

- Now, Open up the `keybindings.json` file for VSCode. Again this can be done by hitting `Ctrl + Shift + P` then searching for and clicking on `Preferences: Open Keyboard Shortcuts (JSON)`
![Screenshot_20230222_021730](https://user-images.githubusercontent.com/34871260/220631902-deed7146-f06d-44f9-9b8b-8d1dfa58cd0f.png)
- Finally add, the following JSON Object entry to the `keybindings.json` file.
```
{
  "key": "Ctrl+Shift+Alt+1",
  "command": "extension.multiCommand.execute",
  "when": "explorerViewletVisible && explorerViewletFocus && explorerResourceIsFolder",
  "args": {
    "sequence": [
      "copyRelativeFilePath",
      "macros.createAureliaComponentFolderAndFiles"
    ]
  }
}
```
This binds the shortcut `Ctrl + Shift + Alt + 1` to this command, if it's not what you want, feel free to change the `key` entry to your liking.
The resulting file should be a JSON array with this (and any other custom key bindings you may have).
