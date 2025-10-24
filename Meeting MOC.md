Meeting MOC 

```dataviewjs

// --- Main Configuration ---
const CONFIG = {
    // Set a folder path to limit the search, or leave as "" to search the whole vault.
    FOLDER_PATH: "", 

    // --- Optional Exclusions ---
    // Add any file names, folder paths, or tags you want to exclude.
    // Leave arrays empty `[]` to disable these filters.
    EXCLUDED_FILE_NAMES: [
        
    ],
    EXCLUDED_FOLDER_PATHS: [
        "3. Resources/_Templates","3. Resources/_Templates Hidden",
    ],
    
    // Excluded tag means remove from the table view - (comma separtated list
    EXCLUDED_TAGS: [
	    "#Meeting"
    ],

    // --- Optional Inclusions (by Frontmatter Properties) ---
    // Notes MUST have these properties and values to be included.
    // Example: { "type": "meeting", "status": "pending" }
    // Leave as {} to disable this filter.
    MUST_HAVE_PROPERTIES: {
         "type": "Meeting" 
    },
	// --- New Note Button Configuration ---
    NEW_NOTE_BUTTON: {
        LABEL: "New Meeting Note",
        TEMPLATE_FILE: "",
        FOLDER_PATH: "",
        FILE_NAME: "Meeting",
        // Set a color (e.g., "green", "#8A2BE2") or leave as "" for default blue.
        COLOR: "#0088cc" 
    }
};

// --- Script Body (No need to edit below this line) ---

const container = this.container;
container.innerHTML = ''; // Clear on full re-render.

// --- Create a header container with Flexbox for alignment ---
const headerContainer = container.createEl('div');
headerContainer.style.display = 'flex';
headerContainer.style.justifyContent = 'space-between';
headerContainer.style.alignItems = 'center';
headerContainer.style.marginBottom = '10px';
headerContainer.style.gap = '10px'; // Add some space between elements

// --- Create "New Note" Button ---
const newNoteButton = headerContainer.createEl('button', { text: CONFIG.NEW_NOTE_BUTTON.LABEL });
newNoteButton.style.cursor = 'pointer';

// Apply button color styling.
if (CONFIG.NEW_NOTE_BUTTON.COLOR) {
    newNoteButton.style.backgroundColor = CONFIG.NEW_NOTE_BUTTON.COLOR;
    newNoteButton.style.color = 'white'; // Assuming a dark background needs white text.
    newNoteButton.style.border = 'none';
} else {
    // Use Obsidian's default style if no color is specified.
    newNoteButton.classList.add('mod-cta');
}

newNoteButton.onclick = async () => {
    const templatePath = CONFIG.NEW_NOTE_BUTTON.TEMPLATE_FILE;
    const folder = CONFIG.NEW_NOTE_BUTTON.FOLDER_PATH;
    const baseFilename = CONFIG.NEW_NOTE_BUTTON.FILE_NAME;

    // Use Templater if a template file is specified
    if (templatePath) {
        const templater = app.plugins.plugins['templater-obsidian'];
        if (!templater) {
            new Notice("Templater plugin is not enabled, but a template is specified.", 5000);
            return;
        }
        
        const template = app.vault.getAbstractFileByPath(templatePath);
        if (!template) {
            new Notice(`Template not found at: ${templatePath}`, 5000);
            return;
        }

        try {
            await templater.templater.create_new_note_from_template(template, folder, baseFilename);
        } catch (e) {
            console.error("Error creating note from template:", e);
            new Notice("Error creating note from template. Check console for details.", 5000);
        }
    } else {
        // Create a new, empty note if no template is specified
        try {
            let path = `${folder ? folder + '/' : ''}${baseFilename}.md`;
            let file = app.vault.getAbstractFileByPath(path);
            let i = 1;
            // Find a unique filename if the base name already exists
            while(file) {
                path = `${folder ? folder + '/' : ''}${baseFilename} ${i}.md`;
                file = app.vault.getAbstractFileByPath(path);
                i++;
            }

            const newFile = await app.vault.create(path, '');
            
            // Open the new note for the user.
            const newLeaf = app.workspace.getLeaf(true);
            await newLeaf.openFile(newFile);
            
            new Notice(`Created new note: ${newFile.basename}`);
        } catch (e) {
            console.error("Error creating new note:", e);
            new Notice("Error creating new note. Check console for details.", 5000);
        }
    }
};

// Create the dropdown menu in the middle.
const selectElement = headerContainer.createEl('select');
selectElement.style.flexGrow = '1'; // Allow the select box to fill available space.

// Create the count element on the right.
const countElement = headerContainer.createEl('div');
countElement.style.fontWeight = 'bold';
countElement.style.textAlign = 'right';
countElement.style.paddingRight = '40px';

// Determine the initial set of pages.
const initialPages = CONFIG.FOLDER_PATH ? dv.pages(`"${CONFIG.FOLDER_PATH}"`) : dv.pages();

// --- Populate UI Elements ---
const tagsInFolder = new Set();
initialPages
    .where(p => {
        if (CONFIG.EXCLUDED_FILE_NAMES.includes(p.file.name)) return false;
        if (CONFIG.EXCLUDED_FOLDER_PATHS.some(path => p.file.path.startsWith(path))) return false;
        for (const [key, value] of Object.entries(CONFIG.MUST_HAVE_PROPERTIES)) {
            if (p[key] !== value) return false;
        }
        return true;
    })
    .forEach(p => {
        if (p.file.tags) {
            p.file.tags.forEach(tag => {
                if (!CONFIG.EXCLUDED_TAGS.includes(tag)) {
                    tagsInFolder.add(tag.substring(1));
                }
            });
        }
    });

const dropdownOptions = ["-- Show All --", ...Array.from(tagsInFolder).sort()];
for (const tag of dropdownOptions) {
    selectElement.add(container.createEl('option', { text: tag, value: tag }));
}

// --- The Core Drawing Function ---
const drawTable = (filter) => {
    container.querySelectorAll(".table-view-table").forEach(node => {
        if (node.parentNode) node.parentNode.remove();
    });

    const pages = initialPages
        .where(p => {
            if (CONFIG.EXCLUDED_FILE_NAMES.includes(p.file.name)) return false;
            if (CONFIG.EXCLUDED_FOLDER_PATHS.some(path => p.file.path.startsWith(path))) return false;
            for (const [key, value] of Object.entries(CONFIG.MUST_HAVE_PROPERTIES)) {
                if (p[key] !== value) return false;
            }
            if (!filter || filter === "-- Show All --") return true;
            return p.file.tags && p.file.tags.some(t => t.substring(1) === filter);
        })
        .sort(p => p.file.name, 'desc');

    countElement.setText(`Notes Found: ${pages.length}`);

    const rows = pages.map(p => {
        const displayTags = p.file.tags ? p.file.tags.filter(t => !CONFIG.EXCLUDED_TAGS.includes(t)) : [];
        const formattedDate = p.file.cday ? p.file.cday.toFormat("dd/MM/yyyy") : "No date";
        return [p.file.link, formattedDate, displayTags.join(", ")];
    });
    
    dv.table(["File", "Date", "Tags"], rows, container);
}

// --- Event Handling and Initial Render ---
selectElement.onchange = () => drawTable(selectElement.value);
drawTable(selectElement.value);

```
