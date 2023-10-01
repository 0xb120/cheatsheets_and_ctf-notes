# Default settins

```json
# For documentation, see https://www.sumatrapdfreader.org/settings/settings3-4-6.html
# customization options for PDF, XPS, DjVu and PostScript UI
FixedPageUI [
    color value with which black (text) will be substituted
    TextColor = #000000

    color value with which white (background) will be substituted
    BackgroundColor = #ffffff

    color value for the text selection rectangle (also used to highlight found text) (introduced in
    version 2.4)
    SelectionColor = #f5fc0c

    top, right, bottom and left margin (in that order) between window and document
    WindowMargin = 2 4 2 4

    horizontal and vertical distance between two pages in facing and book view modes
    PageSpacing = 4 4

    colors to use for the gradient from top to bottom (stops will be inserted at regular intervals
    throughout the document); currently only up to three colors are supported; the idea behind this
    experimental feature is that the background might allow to subconsciously determine reading
    progress; suggested values: #2828aa #28aa28 #aa2828
    GradientColors =

    if true, hides the scrollbars but retains ability to scroll
    HideScrollbars = false
]

customization options for Comic Book and images UI
ComicBookUI [
    top, right, bottom and left margin (in that order) between window and document
    WindowMargin = 0 0 0 0

    horizontal and vertical distance between two pages in facing and book view modes
    PageSpacing = 4 4

    if true, default to displaying Comic Book files in manga mode (from right to left if showing 2
    pages at a time)
    CbxMangaMode = false
]

customization options for CHM UI. If UseFixedPageUI is true, FixedPageUI settings apply instead
ChmUI [
    if true, the UI used for PDF documents will be used for CHM documents as well
    UseFixedPageUI = false
]

list of handlers for selected text, shown in context menu when text selection is active. See docs for more information
SelectionHandlers [
  [
    url to invoke for the selection. ${selection} will be replaced with current selection and
    ${userlang} with language code for current UI (e.g. 'de' for German)
    URL =

    name shown in context menu
    Name =
  ]
]

list of additional external viewers for various file types. See docs for more information
ExternalViewers [
  [
    command line with which to call the external viewer, may contain %p for page number and
    "%1" for the file name (add quotation marks around paths containing spaces)
    CommandLine =

    name of the external viewer to be shown in the menu (implied by CommandLine if missing)
    Name =

    optional filter for which file types the menu item is to be shown; separate multiple entries
    using ';' and don't include any spaces (e.g. *.pdf;*.xps for all PDF and XPS
    documents)
    Filter =
  ]
]

sequence of zoom levels when zooming in/out; all values must lie between 8.33 and 6400
ZoomLevels = 8.33 12.5 18 25 33.33 50 66.67 75 100 125 150 200 300 400 600 800 1000 1200 1600 2000 2400 3200 4800 6400

zoom step size in percents relative to the current zoom level. if zero or negative, the values from
ZoomLevels are used instead
ZoomIncrement = 0

these override the default settings in the Print dialog
PrinterDefaults [
    default value for scaling (shrink, fit, none)
    PrintScale = shrink
]

customization options for how we show forward search results (used from LaTeX editors)
ForwardSearch [
    when set to a positive value, the forward search highlight style will be changed to a rectangle
    at the left of the page (with the indicated amount of margin from the page margin)
    HighlightOffset = 0

    width of the highlight rectangle (if HighlightOffset is > 0)
    HighlightWidth = 15

    color used for the forward search highlight
    HighlightColor = #6581ff

    if true, highlight remains visible until the next mouse click (instead of fading away
    immediately)
    HighlightPermanent = false
]

default values for annotations in PDF documents (introduced in version 3.3)
Annotations [
    color used for highlight annotations
    HighlightColor = #ffff00

    color used for underline annotations
    UnderlineColor = #00ff00

    color used for text icon annotation
    TextIconColor = #ffff00

    type of text annotation icon: comment, help, insert, key, new paragraph, note, paragraph. If not
    set: note.
    TextIconType = 

    default author for created annotations, use (none) to not add an author at all. If not set will
    use Windows user name (introduced in version 3.4)
    DefaultAuthor = 
]

a whitespace separated list of passwords to try when opening a password protected document
(passwords containing spaces must be quoted) (introduced in version 2.4)
DefaultPasswords =

if true, we remember which files we opened and their display settings
RememberOpenedFiles = true

if true, we store display settings for each document separately (i.e. everything after
UseDefaultState in FileStates)
RememberStatePerDocument = true

if true and SessionData isn't empty, that session will be restored at startup
RestoreSession = true

ISO code of the current UI language
UiLanguage =

pattern used to launch the LaTeX editor when doing inverse search
InverseSearchCmdLine =

if true, we expose the SyncTeX inverse search command line in Settings -> Options
EnableTeXEnhancements = false

default layout of pages. valid values: automatic, single page, facing, book view, continuous,
continuous facing, continuous book view
DefaultDisplayMode = automatic

default zoom (in %) or one of those values: fit page, fit width, fit content
DefaultZoom = fit page

custom keyboard shortcuts
Shortcuts [
  [
    command
    Cmd = 

    keyboard shortcut (e.g. Ctrl-Alt-F)
    Key = 
  ]
]

if true, Esc key closes SumatraPDF
EscToExit = false

if true, we'll always open files using existing SumatraPDF process
ReuseInstance = false

if true, a document will be reloaded automatically whenever it's changed (currently doesn't
work for documents shown in the ebook UI) (introduced in version 2.5)
ReloadModifiedDocuments = true

background color of the non-document windows, traditionally yellow
MainWindowBackground = #80fff200

if true, we show the full path to a file in the title bar (introduced in version 3.0)
FullPathInTitle = false

if false, the menu bar will be hidden for all newly opened windows (use F9 to show it until the
window closes or Alt to show it just briefly), only applies if UseTabs is false (introduced in
version 2.5)
ShowMenubar = true

if true, we show the toolbar at the top of the window
ShowToolbar = true

if true, we show the Favorites sidebar
ShowFavorites = false

if true, we show table of contents (Bookmarks) sidebar if it's present in the document
ShowToc = true

if both favorites and bookmarks parts of sidebar are visible, this is the height of bookmarks (table
of contents) part
TocDy = 0

width of favorites/bookmarks sidebar (if shown)
SidebarDx = 0

height of toolbar (introduced in version 3.4)
ToolbarSize = 18

maximum width of a single tab
TabWidth = 300

font size for bookmarks and favorites tree views. 0 means Windows default (introduced in version
3.3)
TreeFontSize = 0

if true, implements smooth scrolling
SmoothScroll = false

if true, we show a list of frequently read documents when no document is loaded
ShowStartPage = true

if true, we check once a day if an update is available
CheckForUpdates = true

we won't ask again to update to this version
VersionToSkip =

default state of the window. 1 is normal, 2 is maximized, 3 is fullscreen, 4 is minimized
WindowState = 1

default position (x, y) and size (width, height) of the window
WindowPos = 0 0 0 0

if true, documents are opened in tabs instead of new windows (introduced in version 3.0)
UseTabs = true

if true, we use Windows system colors for background/text color. Over-rides other settings
UseSysColors = false

actual resolution of the main screen in DPI (if this value isn't positive, the system's UI
setting is used) (introduced in version 2.5)
CustomScreenDPI = 0

information about opened files (in most recently used order)
FileStates [
  [
    path of the document
    FilePath =

    Values which are persisted for bookmarks/favorites
    Favorites [
      [
        name of this favorite as shown in the menu
        Name =

        number of the bookmarked page
        PageNo = 0

        label for this page (only present if logical and physical page numbers are not the same)
        PageLabel =
      ]
    ]

    a document can be "pinned" to the Frequently Read list so that it isn't displaced by
    recently opened documents
    IsPinned = false

    if true, the file is considered missing and won't be shown in any list
    IsMissing = false

    number of times this document has been opened recently
    OpenCount = 0

    data required to open a password protected document without having to ask for the password again
    DecryptionKey =

    if true, we use global defaults when opening this file (instead of the values below)
    UseDefaultState = false

    layout of pages. valid values: automatic, single page, facing, book view, continuous, continuous
    facing, continuous book view
    DisplayMode = automatic

    how far this document has been scrolled (in x and y direction)
    ScrollPos = 0 0

    number of the last read page
    PageNo = 1

    zoom (in %) or one of those values: fit page, fit width, fit content
    Zoom = fit page

    how far pages have been rotated as a multiple of 90 degrees
    Rotation = 0

    state of the window. 1 is normal, 2 is maximized, 3 is fullscreen, 4 is minimized
    WindowState = 0

    default position (can be on any monitor)
    WindowPos = 0 0 0 0

    if true, we show table of contents (Bookmarks) sidebar if it's present in the document
    ShowToc = true

    width of the left sidebar panel containing the table of contents
    SidebarDx = 0

    if true, the document is displayed right-to-left in facing and book view modes (only used for
    comic book documents)
    DisplayR2L = false

    data required to restore the last read page in the ebook UI
    ReparseIdx = 0

    data required to determine which parts of the table of contents have been expanded
    TocState =
  ]
]

state of the last session, usage depends on RestoreSession (introduced in version 3.1)
SessionData [
  [
    data required for restoring the view state of a single tab
    TabStates [
      [
        path of the document
        FilePath =

        same as FileStates -> DisplayMode
        DisplayMode = automatic

        number of the last read page
        PageNo = 1

        same as FileStates -> Zoom
        Zoom = fit page

        same as FileStates -> Rotation
        Rotation = 0

        how far this document has been scrolled (in x and y direction)
        ScrollPos = 0 0

        if true, the table of contents was shown when the document was closed
        ShowToc = true

        same as FileStates -> TocState
        TocState =
      ]
    ]

    index of the currently selected tab (1-based)
    TabIndex = 1

    same as FileState -> WindowState
    WindowState = 0

    default position (can be on any monitor)
    WindowPos = 0 0 0 0

    width of favorites/bookmarks sidebar (if shown)
    SidebarDx = 0
  ]
]

data required for reloading documents after an auto-update (introduced in version 3.0)
ReopenOnce =

data required to determine when SumatraPDF last checked for updates
TimeOfLastUpdateCheck = 0 0

value required to determine recency for the OpenCount value in FileStates
OpenCountWeek = 0
```

# Custom settings

```json
# For documentation, see https://www.sumatrapdfreader.org/settings/settings3-4-6.html
FixedPageUI [
	TextColor = #eeeeee
	BackgroundColor = #171717
	SelectionColor = #f5fc0c
	WindowMargin = 2 4 2 4
	PageSpacing = 4 4
	GradientColors = #333333
	HideScrollbars = false
]
ComicBookUI [
	WindowMargin = 0 0 0 0
	PageSpacing = 4 4
	CbxMangaMode = false
]
ChmUI [
	UseFixedPageUI = false
]

SelectionHandlers [
]
ExternalViewers [
]

ZoomLevels = 8.33 12.5 18 25 33.33 50 66.67 75 100 125 150 200 300 400 600 800 1000 1200 1600 2000 2400 3200 4800 6400
ZoomIncrement = 0

PrinterDefaults [
	PrintScale = shrink
]
ForwardSearch [
	HighlightOffset = 0
	HighlightWidth = 15
	HighlightColor = #6581ff
	HighlightPermanent = false
]
Annotations [
	HighlightColor = #ffff00
	UnderlineColor = #00ff00
	TextIconColor = #ffff00
	TextIconType = 
	DefaultAuthor = 
]

RememberOpenedFiles = true
RememberStatePerDocument = true
RestoreSession = true
UiLanguage = en
EnableTeXEnhancements = false
DefaultDisplayMode = automatic
DefaultZoom = fit page
Shortcuts [
]
EscToExit = false
ReuseInstance = false
ReloadModifiedDocuments = true

MainWindowBackground = #80fff200
FullPathInTitle = false
ShowMenubar = true
ShowToolbar = true
ShowFavorites = false
ShowToc = true
TocDy = 0
SidebarDx = 427
ToolbarSize = 18
TabWidth = 300
TreeFontSize = 0
SmoothScroll = false
ShowStartPage = true
CheckForUpdates = true
WindowState = 2
WindowPos = 354 390 1180 1032
UseTabs = true
UseSysColors = false
CustomScreenDPI = 0

FileStates [
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\Offensive Security AWAE (OSWE) - 2020\Advanced Web Attacks and Exploitation 2019 (AWAE).pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 1
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = false
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\The Web Application Hacker Handbook.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 2
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 327.919
		PageNo = 138
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = true
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
		TocState = 5 59 60 66 100 426 430 436
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\OWASP_Code_Review_Guide_v2.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 1
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = false
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\Mastering_Burp_Suite_Pro.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 2
		UseDefaultState = false
		DisplayMode = single page
		ScrollPos = -1 -1
		PageNo = 731
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = true
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
		TocState = 89 97 101 106
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\eLearnSecurity WAPTxv2 (eWPTXv2) - 2020\PDF\116_Advanced_Web_Application_Exploitation hide01.ir.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 61
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = false
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\eLearnSecurity WAPTxv2 (eWPTXv2) - 2020\PDF\Advanced Web Application Exploitation hide01.ir.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 61
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = false
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\eLearnSecurity WAPTxv2 (eWPTXv2) - 2020\PDF\1. Encoding and Filtering hide01.ir.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = false
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\SEC642 - Advanced Web App Penetration Testing, Ethical Hacking, and Exploitation Techniques (2017)\SEC642.2 - Web Frameworks.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 1
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 473.758
		PageNo = 104
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = false
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\SEC642 - Advanced Web App Penetration Testing, Ethical Hacking, and Exploitation Techniques (2017)\SEC642.1 - Advanced Attacks.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 42
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 354 390 1180 1032
		ShowToc = false
		SidebarDx = 427
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\OWASP_MASTG.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 1
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 1331 -1096 1180 1032
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\eLearnSecurity WAPTxv2 (eWPTXv2) - 2020\PDF\4. XSS - Filter Evasion and WAF Bypassing hide01.ir.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 5.27833
		PageNo = 17
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 1331 -1096 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Web\eWPTX\WAPTx sec 2.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\SEC542 - Web App Penetration Testing and Ethical Hacking (2017)\PDFs\2016\SEC542_Book6.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 4
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\SEC542 - Web App Penetration Testing and Ethical Hacking (2017)\PDFs\2016\SEC542_Book5.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 48.8862
		PageNo = 150
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\SEC642 - Advanced Web App PT, Ethical Hacking, and Exploitation (2016, 2017)\2017 - PDF & VM\PDF\SEC642.6 - Capture the Flag.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\SEC642 - Advanced Web App PT, Ethical Hacking, and Exploitation (2016, 2017)\2017 - PDF & VM\PDF\SEC642.5 - Web Application Firewall and Filter Bypass.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\SEC642 - Advanced Web App PT, Ethical Hacking, and Exploitation (2016, 2017)\2017 - PDF & VM\PDF\SEC642.4 - Alternative Web Interfaces.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 497.93
		PageNo = 138
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\SEC642 - Advanced Web App PT, Ethical Hacking, and Exploitation (2016, 2017)\2017 - PDF & VM\PDF\SEC642.3 - Web Cryptography.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 640.799
		PageNo = 3
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\eLearnSecurity eWPT Notes.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 122.564
		PageNo = 327
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Mobile App Penetration Testing\Android Hackers Handbook.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous book view
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Mobile App Penetration Testing\eLearnSecurity Mobile Application Penetration Testing Notes by Joas.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 11
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Mobile App Penetration Testing\Learning Pentesting for Android Devices.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Mobile App Penetration Testing\The Mobile Application Hacker’s Handbok (2015).pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 206.556
		PageNo = 237
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Mobile App Penetration Testing\Mobile Application Penetration Testing (2016).pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 224
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\eWPTX\WAPTx sec 9.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\eWPTX\WAPTx sec 8.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\eWPTX\WAPTx sec 1.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 1
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 800 -1049 1180 1032
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\SEC542 - Web App Penetration Testing and Ethical Hacking (2017)\SEC542_Book3.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 4
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\SEC542 - Web App Penetration Testing and Ethical Hacking (2017)\SEC542_Book1.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 6
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\Android_app_vulnerability_classes.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 2
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\EXP-301 - Windows User Mode Exploit Development (OSED) hide01.ir.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\The car hackers handbook.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Documents\VM-Shared\privatevault\report.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = single page
		ScrollPos = -1 -1
		PageNo = 28
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Downloads\RH2023-T7JRU-1-pdf (1).pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Desktop\TO READ\eWPTX\WAPTx sec 6.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 103.575
		PageNo = 90
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Downloads\Telegram Desktop\Cacti - CVE-2022-46169 - start.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 371.305
		PageNo = 4
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\Second Brain\zzz_res\attachments\20220520090835.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Downloads\Cacti_-_CVE-2022-46169.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 424.613
		PageNo = 4
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = G:\My Drive\9d18da5a7c526c9b87d3f10fe6b93631706958e9f2ec701c3d5a3c4d1ad0338b.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 1
		WindowPos = 690 0 1180 1528
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Downloads\OWASP_MASVS.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 21
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = true
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Downloads\Consiglio1692098 (1).pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = C:\Users\mbrol\Downloads\RH2023-T7JRU-1-pdf.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 1
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = ?:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Mobile App Penetration Testing\eMAPT - Mobile Application Penetration Testing Professional\PDF\Android Mobile App Pentesting\11. Dynamic Code Analysis hide01.ir.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 124.723
		PageNo = 30
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
	[
		FilePath = E:\0xbro\Books & Manuals\Penetration Testing & Red Teaming\Web App Penetration Testing\AWAE PDF 2020\AWAE 2020 hide01.ir.pdf
		Favorites [
		]
		IsPinned = false
		IsMissing = false
		OpenCount = 0
		UseDefaultState = false
		DisplayMode = continuous
		ScrollPos = -1 -1
		PageNo = 445
		Zoom = fit page
		Rotation = 0
		WindowState = 2
		WindowPos = 823 -1099 1180 1528
		ShowToc = false
		SidebarDx = 289
		DisplayR2L = false
		ReparseIdx = 0
	]
]
SessionData [
]
TimeOfLastUpdateCheck = 0 0
OpenCountWeek = 653

# Settings below are not recognized by the current version
```