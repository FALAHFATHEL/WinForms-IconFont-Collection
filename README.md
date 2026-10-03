# 🎨 WinForms IconFont Collection

> A high-DPI friendly icon font library for Windows Forms, with `IconFontManager` for simple in-memory font loading and UI integration.

<p align="center">
<img src="https://img.shields.io/badge/.NET%20Framework-4.7.2%2B-512BD4?logo=dotnet&logoColor=white" alt=".NET Framework 4.7.2+" />

<img src="https://img.shields.io/badge/Icons-Vector%20%F0%9F%8E%AF%20DPI%20Aware-important" alt="Vector Icons DPI Aware" />
<img src="https://img.shields.io/badge/Memory-Safe%20%F0%9F%A7%A0-green" alt="Memory Safe" />
<img src="https://img.shields.io/badge/Zero-Installation%20%F0%9F%93%A6-blue" alt="Zero Installation" />
<img src="https://img.shields.io/badge/DPI-Scaling%20Ready%20%F0%9F%96%A5%EF%B8%8F-ff69b4" alt="DPI Scaling Ready" />

<img src="https://img.shields.io/github/stars/FALCONzeroX/WinForms-IconFont-Collection?style=social" alt="GitHub Stars" />
<img src="https://img.shields.io/github/forks/FALCONzeroX/WinForms-IconFont-Collection?style=social" alt="GitHub Forks" />
<img src="https://img.shields.io/github/issues/FALCONzeroX/WinForms-IconFont-Collection" alt="GitHub Issues" />
<img src="https://img.shields.io/github/last-commit/FALCONzeroX/WinForms-IconFont-Collection" alt="Last Commit" />

<img src="https://img.shields.io/badge/Icon%20Sets-Font%20Awesome%20%7C%20Material%20%7C%20Ionicons-informational" alt="Supported Icon Sets" />

<img src="https://img.shields.io/badge/build-passing-brightgreen" alt="Build Status" />
<img src="https://img.shields.io/badge/code%20quality-A+-brightgreen" alt="Code Quality" />
</p>

<p align="center">
  <img src="Poster.png" alt="Icon Collection Showcase" width="100%" style="border-radius: 10px;" />
</p>

---

## The Problem: Why Traditional Icons Fail on Modern Screens

If you’ve ever built a Windows Forms application and tested it on a high‑DPI monitor (4K, 125%, 150%, 200% scaling), you’ve witnessed the **icon blur**. A beautifully designed 32×32 pixel icon becomes a smeared, fuzzy mess the moment the operating system tries to stretch it to match the screen’s scaling factor.

This isn’t a bug in your code – it’s a fundamental limitation of **raster (bitmap) image formats** like `.png` and `.ico`.

### How Bitmap Icons Work (and Why They Break)

A PNG or ICO file stores an icon as a fixed grid of colored pixels. At 100% DPI (96 DPI standard), every pixel maps perfectly to a physical screen pixel. But modern screens have much higher pixel densities. To keep UI elements physically the same size, Windows tells applications to render at a larger logical scale – for example, **150%** means every control (and every icon) must be drawn 1.5× larger.

With a bitmap, there is no extra detail beyond the original grid. The graphics engine must **interpolate** – guess what the extra pixels should look like. This interpolation creates:

- **Blur and softness**: edges become undefined.
- **Jagged “stair‑step” artifacts**: diagonal lines look pixelated.
- **Inconsistent appearance**: icons designed at 16×16 look completely different from those forced to render at 24×24 or 32×32.

Even the common “multi‑resolution ICO” trick (embedding several fixed sizes like 16×16, 32×32, 48×48) fails at fractional scaling values like 125% or 175%, because the scaling factor rarely matches an exact pre‑made size. The result is always a compromise.

---

## Why Icon Fonts Are the Superior Choice

Icon fonts (`.ttf` / `.otf`) solve every scaling problem mentioned above **by design**. Instead of storing pixels, they store **mathematical outlines** – just like the system fonts you use for text (Segoe UI, Arial, etc.).

### The Vector Advantage

- **Infinite resolution**: A vector outline is resolution‑independent. When you render it at 16pt, 64pt, or 256pt, the operating system recalculates the exact shape using the same mathematical curves. The result is **perfect sharpness at any DPI level** – no interpolation, no blur.
- **Sub‑pixel rendering**: Modern font engines apply anti‑aliasing (ClearType) that leverages the physical arrangement of LCD sub‑pixels, making icon edges even smoother and more readable than a bitmap could ever achieve.
- **Consistency across the entire application**: Because icon fonts are rendered by the same text layout engine as labels and buttons, they automatically respect the system’s DPI settings. You set a font size in points, and the system handles the rest.
- **Scalability (Flexibility when resizing)**: Raster images (such as PNG or ICO files) come in fixed sizes; if you try to scale them up or down, they do not adapt well because their dimensions are predetermined. In contrast, using an icon font offers complete flexibility during application development, as the icons are treated just like standard characters.

---

## 🔒 Font Deployment & Resource Embedding

When you use a custom icon font in a Windows Forms application, you must deliver the font file (`.ttf` / `.otf`) to every user’s machine so that the operating system can render the icons. How you deliver it directly impacts security, reliability, and maintenance.

### The Problem

Fonts are normally stored in `C:\Windows\Fonts` and are available to all applications. To make a custom font available, you have three options:

| Approach                         | Problems                                                                                                                                                                                                                                                        |
| :------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Install font globally**     | - Requires administrator rights (UAC prompt).<br>- Pollutes the system font folder permanently.<br>- May conflict with other versions of the same font.<br>- Uninstalling your app does not remove the font.<br>- Breaks on locked‑down corporate environments. |
| **2. Ship font as a loose file** | - The `.ttf` must be placed next to the `.exe` – easy to delete or lose.<br>- Network deployments and shortcuts break if the file is missing.<br>- Extra file means extra support headaches.                                                                    |
| **3. Embed font as resource**    | ✅ **No admin rights needed.**<br>✅ **Single‑file deployment** – font lives inside the `.exe`.<br>✅ **Impossible to lose or misplace.**<br>✅ **Isolated** – no other app can see or interfere with it.<br>✅ **Clean uninstall** – nothing left behind.      |

### Why Embedding as a Resource Works Well

Embedding the font as a **binary resource** inside your `.exe` works by loading the font data directly into unmanaged memory and registering it with GDI+ via `PrivateFontCollection.AddMemoryFont`. The font exists **only in the memory of your process** and is completely invisible to the rest of the system. This technique:

- **Bypasses the need for administrator elevation** – no setup, no UAC pop‑ups.
- **Guarantees the exact version of the font** you tested with – no risk of a user replacing it with an incompatible copy.
- **Enables true single‑file deployments** (xcopy, ClickOnce, MSIX) with no external dependencies.
- **Keeps the user’s system clean** – the font disappears when the application exits.

> ⚠️ **Critical implementation note:** The memory block holding the font data must not be freed until the `PrivateFontCollection` is disposed. Premature cleanup causes rendering crashes. Our architecture uses **deferred cleanup** (tracking all `IntPtr` pointers and freeing them only on application exit) to guarantee safety.

### Practical Benefits for WinForms Developers

- **Zero installation on the client machine**: By embedding the font file directly inside your `.exe` as a resource, you bypass the need to install anything in the Windows Fonts folder. The font is loaded entirely from memory and stays private to your application.
- **Single‑file deployment**: The icon set travels with your executable – no extra PNG files to lose or mismanage.
- **Flexibility**: You can change the icon color, size, or even apply text effects (bold, outline) just by modifying the `Font` object’s properties. No need to ask a designer for a new asset file.

---

# 🚀 How to Use `IconFontManager`

The project now includes an `IconFontManager` class that handles the font lifecycle for you.  
You no longer need to write `PrivateFontCollection`, `Marshal`, `AddFontMemResourceEx`, pointer tracking, or manual font cleanup code inside your Forms.

### ✨ The Basic Idea

The workflow is only:

1. Add the `.ttf` font to **Resources**.
2. Create **`IconFontManager`**.
3. Apply it to **controls**
4. Set the icon to control
5. Dispose the manager.

---

# 🛠️ Step 1: Add the Font to Project Resources

The font must be included in your application resources so it can be loaded as a `byte[]`.

1. Open **Solution Explorer**.
2. Right-click your project → **Properties**.
3. Open **Resources**.
4. Choose **Add Resource → Choose Type 'FILE' then Select File…**
5. Select the `.ttf` or `.otf` file From The Icon Font in this repo Then Click Add.

> 💡 **Tip:** Keep the resource name simple, for example `Design_icons`, `System_icons`, or `BRANDS_Icons`.

---

# 💻 Step 2: Create the `IconFontManager`

Add the class to your project and import its namespace:

1. Open **Solution Explorer**.
2. Right-click your project Then → **Add** → **Existing Item...**
3. Select The Class File `IconFontManager.cs`
4. Now The Class is in your project
5. Go into `Form1.cs` or Any Form You Need To Apply Font On And ADD Namespace IFM (Icon Font Manager)

```csharp
using IFM;
```

The manager internally keeps track of the loaded fonts and their resources.

Then create one manager for the Form and initialize it in Counstructor:

> 💡 **Tip:** It's better to create a const variable for Font Name to avoid forgetting font name Like:

```csharp
private const string FONT_KEY_DEVICES = "Devices";
```

---

# ➕ Step 3: Appling The Font

### 1. Create an object of IconFontManager

```csharp
private readonly IconFontManager _fontManager = new IconFontManager();
```

### 2. Create constant variable for font key name

```csharp
private const string FONT_KEY = "Devies";
```

### 3. Create a LoadForm Event then apply font in it

```csharp
_fontManager.RegisterAndApply(FONT_KEY, Properties.Resources.Devices_icons, 20f, this);
             //Parameters => (FONT_KEY, Bytes Array(Font), Start Size, CuurentInterface)
```

```csharp
private void Form1_Load(object sender, EventArgs e)
{
    _fontManager.RegisterAndApply(FONT_KEY, Properties.Resources.Devices_icons, 20f, this);

    // Apply Icon on controls
    SetupFormIcons();
}
```

### 4. Apply Icon On Controls using SetIconWithText() and SetIcon() Functions

```csharp
private void SetupFormIcons()
{
    // Set Icon With Text (Not Preffered)
    _fontManager.SetIconWithText(btnSave, FONT_KEY, DevicesIcons.Save2Fill, "Save Data");

    // Set Icon With no text
    _fontManager.SetIcon(btnClose, FONT_KEY, DevicesIcons.CastLine);

    // Set Icon With a specific font size
    _fontManager.SetIcon(lblStatus, FONT_KEY, DevicesIcons.WIFI_Line, 40f);
}
```

> 💡 **Note:** DevicesIcons is a class that has a const icon unicode variable to facilitate the process of finding icons

```csharp
public static class DevicesIcons
{
    public const string BarcodeBoxFill = "\ue900";
    public const string AirplayFill = "\ue901";
    public const string AirplayLine = "\ue902";
    public const string BarcodeBoxLine = "\ue903";
    public const string BarcodeFill = "\ue904";
    public const string BarcodeLine = "\ue905";
    public const string BaseStationFill = "\ue906";
    //..............
}
```

### 5. Free Memory

```csharp
protected override void OnFormClosed(FormClosedEventArgs e)
{
    base.OnFormClosed(e);
    _fontManager?.Dispose();
}
```

### `Form1.cs`

```csharp
public partial class Form1 : Form
{
    private readonly IconFontManager _fontManager = new IconFontManager();
    private const string FONT_KEY = "Devies";
    public Form1()
    {
        InitializeComponent();
    }

    private void Form1_Load(object sender, EventArgs e)
    {
        _fontManager.RegisterAndApply(FONT_KEY, Properties.Resources.Devices_icons, 20f, this);

        // Apply Icon on controls
        SetupFormIcons();
    }

    private void SetupFormIcons()
    {
        // Set Icon With Text (Not Preffered)
        _fontManager.SetIconWithText(btnSave, FONT_KEY, DevicesIcons.Save2Fill, "Save Data");

        // Set Icon With no text
        _fontManager.SetIcon(btnClose, FONT_KEY, DevicesIcons.CastLine);

        // Set Icon With a specific font size
        _fontManager.SetIcon(lblStatus, FONT_KEY, IconCodes.WIFI_Line, 40f);
    }

    // Free Memory
    protected override void OnFormClosed(FormClosedEventArgs e)
    {
        base.OnFormClosed(e);
        _fontManager?.Dispose();
    }
}
```

# 🎯 Using Scenarios

## Scenario 1: Application-Wide Centralized Setup (Program.cs)

For multi-form applications, register the font once at application startup to optimize memory usage

```csharp
internal static class Program
{
    public static IconFontManager FontManager { get; private set; }
    public const string FONT_KEY = "Devices";

    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);

        // Global Instance
        FontManager = new IconFontManager();
        FontManager.AddFont(FONT_KEY, Properties.Resources.Devices_icons, 12f);

        Application.Run(new Form1());

        // Dispose on shutdown
        FontManager.Dispose();
    }
}
```
Then apply it inside any Form:

```csharp
private void Form2_Load(object sender, EventArgs e)
{
    Program.FontManager.ApplyToAll(this, Program.FONT_KEY);
    Program.FontManager.SetIconWithText(btnDelete, Program.FONT_KEY, IconCodes.Delete, "Delete Item");
}
```

## Scenario 2: Apply Font by Control Type
Target specific control types (e.g., all buttons in a form) without affecting labels or textboxes:
```csharp
_fontManager.ApplyToType<Button>(this, FONT_KEY);
```
## Scenario 3: Conditional Font Application (Predicate)
Apply icon fonts to controls matching a custom condition:
```csharp
// Apply only to controls starting with "btnIcon_"
_fontManager.ApplyWhere(this, FONT_KEY, ctrl => ctrl.Name.StartsWith("btnIcon_"));
```

# 🎯 API Reference

| Method                                     | Description                                                               |
| ------------------------------------------ | ------------------------------------------------------------------------- |
| AddFont(key, bytes, size)                  | Loads font data into memory and registers it under a unique key.          |
| GetFont(key)                               | Safely retrieves the Font object for a given key (null if not found).     |
| HasFont(key)                               | Returns true if the font key is currently loaded in memory.               |
| RegisterAndApply(key, bytes, size, parent) | Registers font and applies it to all child controls of parent.            |
| ApplyToAll(parent, key)                    | Recursively applies font to all child controls under parent.              |
| ApplyToType<T>(parent, key)                | Applies font to all controls of type T within parent.                     |
| ApplyWhere(parent, key, predicate)         | Applies font to controls satisfying a conditional function.               |
| SetIcon(control, key, code)                | Sets standalone unicode icon on a control safely.                         |
| SetIconWithText(control, key, code, text)  | Combines unicode icon and text string on a control safely.                |
| SetIcon(control, key, code, size)          | Sets unicode icon with a custom font size override on a control.          |
| Dispose()                                  | Releases all PrivateFontCollection handles and unmanaged memory pointers. |

---

# 📦 Available Icon Font Packs | حزم خطوط الأيقونات المتوفرة

This repository contains a massive, well-categorized library of custom vector icon fonts. Each pack is meticulously structured to include the font files along with an interactive HTML index for easy search and preview.

يحتوي هذا المستودع على مكتبة ضخمة ومنظمة من خطوط الأيقونات الشعاعية. تم ترتيب كل حزمة لتشمل ملفات الخطوط بالإضافة إلى ملف HTML تفاعلي لتسهيل البحث واستعراض الأيقونات.

---

## 📊 Collection Overview & Statistics | نظرة عامة وإحصائيات الحزم

The collection contains over **9,000+ scalable vector icons** divided into **26 specialized categories** to fit any user interface design required in your desktop software.

تضم المجموعة أكثر من **9,000 أيقونة شعاعية** قابلة للتكبير والتصغير، مقسمة إلى **26 تصنيفاً متخصصاً** لتناسب كافة احتياجات واجهات المستخدم في برامجك.

### 🗂️ Detailed Directory & Icon Counts | تفاصيل المجلدات وأعداد الأيقونات

| #   | Package Name (اسم الحزمة)         | Total Icons (عدد الأيقونات) | Directory Name (اسم المجلد)     |
| --- | --------------------------------- | --------------------------- | ------------------------------- |
| 1   | **FALCON Icon Collection Pack 1** | 2001                        | `FALCON_Icons_Collection_Pack1` |
| 2   | **General Icons Filled Pack 3**   | 1053                        | `Gereral_Icons_Filled_Pack3`    |
| 3   | **FALCON Icon Collection Pack 2** | 699                         | `FALCON_Icon_Collection_Pack2`  |
| 4   | **Brands Icon Pack 1**            | 606                         | `Brands_Icons_Pack1`            |
| 5   | **General Icons PACK 2**          | 562                         | `General_Icons_Pack2`           |
| 6   | **Brands Icon Pack 2**            | 501                         | `Brands_Icons_Pack2`            |
| 7   | **General Icon Pack 1**           | 491                         | `General_Icons_Pack1`           |
| 8   | **System Icons**                  | 348                         | `System_Icons`                  |
| 9   | **Logos Icons**                   | 300                         | `Logos_Icons`                   |
| 10  | **Media Icons**                   | 296                         | `Media_Icons`                   |
| 11  | **Documents Icons**               | 244                         | `Documents_Icons`               |
| 12  | **Design Icons**                  | 236                         | `Design_Icons`                  |
| 13  | **Business Icons**                | 220                         | `Business_Icons`                |
| 14  | **Devices Icons**                 | 192                         | `Devices_Icons`                 |
| 15  | **Arrows Icons**                  | 178                         | `Arrows_Icons`                  |
| 16  | **Finance Icons**                 | 172                         | `Finance_Icons`                 |
| 17  | **Map Icons**                     | 172                         | `Map_Icons`                     |
| 18  | **Editor Icons**                  | 151                         | `Editor_Icons`                  |
| 19  | **Others Icons**                  | 116                         | `Others_Icons`                  |
| 20  | **Communications Icons**          | 92                          | `Communication_Icons`           |
| 21  | **Medical Icons**                 | 84                          | `Medical_Icons`                 |
| 22  | **Weather Icons**                 | 82                          | `Weather_Icons`                 |
| 23  | **Development Icons**             | 66                          | `Development_Icons`             |
| 24  | **Buildings Icons**               | 62                          | `Buildings_Icons`               |
| 25  | **Users Icons**                   | 53                          | `Users_Icons`                   |
| 26  | **Game Sport Icons**              | 50                          | `Game_Sports_Icons`             |
| 📐  | **Total Icons Library**           | **9,061 Icons**             |                                 |

---

# 📂 Folder Structure & Contents | محتويات وهيكلية المجلدات

Each icon package directory in this repository follows a clean, standardized layout generated by IcoMoon. It provides everything you need from raw font files to interactive preview web pages.

يتبع كل مجلد من مجلدات حزم الأيقونات في هذا المستودع هيكلية موحدة ونظيفة (تم إنشاؤها عبر IcoMoon). توفر هذه الهيكلية كل ما تحتاجه بدءاً من ملفات الخطوط الخام وحتى صفحات الاستعراض التفاعلية.

## 🔍 Directory Breakdown | تفصيل محتويات المجلد

Inside every individual folder, you will find the following assets:

داخل كل مجلد فرعي، ستجد الملفات والعناصر التالية:

### 1. 📂 `fonts/` (Directory / مجلد ملفات)

- **English:** Contains the core scalable font files. The `.ttf` file inside this folder is the one you will import into Visual Studio's `Resources.resx`.
- **العربية:** يحتوي على ملفات الخطوط الأساسية. ملف الـ `.ttf` الموجود داخل هذا المجلد هو الملف الذي ستقوم بتضمينه داخل موارد مشروعك `Resources.resx`.

### 2. 🌐 `[Package_Name].html` (Interactive Document / صفحة ويب تفاعلية)

- **English:** The visual index/cheat-sheet for the specific pack (e.g., `Arrows_Icons.html`). Open this in any browser to search, filter, and copy the precise Unicode points for your C# code.
- **العربية:** الفهرس المرئي المخصص للحزمة (مثل `Arrows_Icons.html`). يمكنك فتحه في أي متصفح للبحث عن الأيقونات، وتصفيتها، ونسخ أكواد الـ Unicode الخاصة بها لاستخدامها في كود C#.

### 3. 📄 `selection.json` (JSON Source File / ملف بيانات المشروع)

- **English:** The metadata configuration file containing the project definition, glyph mappings, and character codes. You can re-import this file back into tools like IcoMoon to edit or expand the font pack later.
- **العربية:** ملف البيانات الوصفية (Metadata) الذي يحتوي على إعدادات المشروع وخريطة الرموز وأكوادها. يمكنك إعادة استيراد هذا الملف في أدوات مثل IcoMoon لتعديل حزمة الخطوط أو توسيعها لاحقاً.

### 4. 🎨 `style.css` (CSS Source File / ملف التنسيق)

- **English:** Contains the CSS rules and class mappings for web projects. While not strictly used in Windows Forms, it serves as a valuable reference for internal font naming conventions.
- **العربية:** يحتوي على قواعد التنسيق وربط الكلاسات المخصصة لمشاريع الويب. بالرغم من عدم استخدامه مباشرة في تطبيقات Windows Forms، إلا أنه يمثل مرجعاً ممتازاً لمعرفة الأسماء البرمجية الداخلية للأيقونات.

### 5. 📂 `demo-files/` (Directory / مجلد ملفات العرض)

- **English:** Internal components and assets required to properly render and style the interactive HTML preview page.
- **العربية:** المكونات والملفات المساعدة المطلوبة لتشغيل وعرض صفحة الـ HTML التفاعلية وتنسيقها بشكل صحيح.

---

---

## 👨‍💻 About the Developer | عن المطور

<p align="center">
  <img src="https://github.com/FALCONzeroX.png" width="120" height="120" style="border-radius: 50%;" alt="FALCONzeroX Profile Picture" />
</p>

<h3 align="center">FALAH FATHEL</h3>

<p align="center">
  <b>Desktop Application Developer & Software Architect</b><br>
  <span>Crafting clean, high-performance, and high-DPI aware Windows Forms applications.</span>
</p>

<p align="center">
  <a href="https://github.com/FALCONzeroX"><img src="https://img.shields.io/badge/GitHub-FALCONzeroX-181717?style=flat-square&logo=github" alt="GitHub Profile" /></a>
  <a href="https://github.com/FALCONzeroX/WinForms-IconFont-Collection/issues"><img src="https://img.shields.io/badge/Support-Open%20an%20Issue-green?style=flat-square&logo=github" alt="Open Issue" /></a>
</p>

---

### 🤝 Contributing

We welcome contributions of any kind – icon pack suggestions, performance improvements, documentation translation, or bug fixes. Please open an issue to discuss your ideas or submit a pull request directly. All contributions are valued, and kind, constructive communication is our standard.

## 📄 License

This project is licensed under the **MIT License**. See the `LICENSE` file for details. You are free to use, modify, and distribute the code in personal or commercial projects, provided the original copyright notice remains intact.

---
