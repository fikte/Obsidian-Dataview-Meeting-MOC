# Obsidian Dataview Meeting-MOC

A customizable `dataviewjs` script for Obsidian that provides an interactive table view of your meeting notes, complete with dynamic filtering and a quick "New Note" button.

## Features

- **Quick Note Creation**: Instantly create a new meeting note from a button, either from a template or as a blank file.
- **Flexible Template Support**: Seamlessly integrates with the Templater plugin but works perfectly without it.
- **Dynamic Tag Filtering**: Use a dropdown menu to filter your notes by any tag.
- **Required Property Filtering**: Enforce that all displayed notes contain specific frontmatter properties (e.g., `type: Meeting`).
- **Comprehensive Exclusions**: Easily hide specific files, folders, or tags from the view.
- **Live Note Count**: Always know how many notes match your current filter.
- **Customizable UI**: Change the new note button's label and color.

## Rendered vuew which this dataviewjs script produces

<img width="751" height="192" alt="image" src="https://github.com/user-attachments/assets/1c97935b-5091-4627-9763-e34c515a4150" />

_Screenshot of the rendered view showing the New Meeting button, dropdown to filter by tags, note count in the table and the table of notes this dataviewjs script has found in the vault_ 

<img width="755" height="157" alt="image" src="https://github.com/user-attachments/assets/1dbd3c27-620b-4feb-990f-9baf23b79f6b" />

_Screenshot showing a filter by the Test2 tag_

## Installation

1.  Make sure you have the [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) installed and enabled.
2.  In Dataview's settings, ensure **"Enable JavaScript Queries"** is turned on.
3.  Create a new note in Obsidian.
4.  Copy the entire script from the previous message and paste it into a `dataviewjs` code block in your note:
````
    ```dataviewjs
    // --- Main Configuration ---
    const CONFIG = {
        // ... (paste the full script here)
    };

    ...

    // --- Event Handling and Initial Render ---
    selectElement.onchange = () => drawTable(selectElement.value);
    drawTable(selectElement.value);
    ```
````
    
5.  When you switch to Reading View, the script will render and display your meeting notes manager.

## Configuration

All settings are managed within the `CONFIG` object at the top of the script.

---

### `FOLDER_PATH`
Limits the search to a specific folder. Leave empty to search your entire vault.
```
// Search only within the "Meetings" folder
FOLDER_PATH: "Meetings",

// Search the whole vault
FOLDER_PATH: "",
```

---

### Exclusions
Use these to hide certain notes from the view.

-   `EXCLUDED_FILE_NAMES`: A list of exact file names to exclude.
    ```
    EXCLUDED_FILE_NAMES: ["Meeting Template.md"],
    ```
-   `EXCLUDED_FOLDER_PATHS`: A list of folder paths to exclude. All notes within these folders will be hidden.
    ```
    EXCLUDED_FOLDER_PATHS: ["3. Resources/_Templates"],
    ```
-   `EXCLUDED_TAGS`: A list of tags to exclude. Notes with these tags will be hidden from the table.
    ```
    EXCLUDED_TAGS: ["#archive", "#draft"],
    ```

---

### `MUST_HAVE_PROPERTIES`
Ensures that only notes with specific frontmatter properties and values are displayed. Leave as `{}` to disable.

```
// Only show notes with `type: Meeting` in their frontmatter
MUST_HAVE_PROPERTIES: {
     "type": "Meeting"
},

// Show all notes regardless of properties
MUST_HAVE_PROPERTIES: {},
```

<img width="743" height="235" alt="image" src="https://github.com/user-attachments/assets/2b49ba54-3634-4b30-9dd4-78b7935de35f" />

_Screenshot show an example note with the type property of Meeting set, also shown is a tag of #Test which appears in the dataveiw table as in the screenshots above_ 


---

### `NEW_NOTE_BUTTON`
Controls the appearance and behavior of the "New Note" button.

```
NEW_NOTE_BUTTON: {
    LABEL: "New Meeting Note",
    TEMPLATE_FILE: "",
    FOLDER_PATH: "Meetings",
    FILE_NAME: "Meeting",
    COLOR: "#0088cc"
}
```
-   `LABEL`: The text that appears on the button.
-   `TEMPLATE_FILE`: **(Optional)** The full path to a Templater template file.
    -   If you provide a path (e.g., `"Templates/Meeting.md"`), the Templater plugin is **required**.
    -   If you leave this as `""`, a **blank note** will be created, and Templater is **not needed**.
-   `FOLDER_PATH`: The folder where new notes will be created. Leave as `""` for the vault root.
-   `FILE_NAME`: The base name for the new file. If a file with this name exists, a number will be appended automatically (e.g., `Meeting 1.md`).
-   `COLOR`: The background color of the button. Leave as `""` to use the default Obsidian button style.

## Troubleshooting

-   **"Templater plugin is not enabled..." Error:** This means you have provided a path in `TEMPLATE_FILE` but do not have Templater installed or enabled. To fix, either install Templater or set `TEMPLATE_FILE: ""`.
-   **Notes Not Appearing:**
    1.  Check that the notes have the properties defined in `MUST_HAVE_PROPERTIES`.
    2.  Ensure the notes are not in a folder listed in `EXCLUDED_FOLDER_PATHS`.
    3.  Make sure the note's file name is not in `EXCLUDED_FILE_NAMES`.
-   **Template Not Found Error:**
    -   Verify the path in `TEMPLATE_FILE` is exactly correct, starting from your vault's root folder.
    -   Ensure the file name and extension are correct.
