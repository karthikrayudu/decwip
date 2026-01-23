# decwip

🔹 TECHNOLOGY STACK (What we will use)


ASP.NET Core MVC (.NET 6 or 7)


SQL Server


Entity Framework Core


Azure Form Recognizer (Document Intelligence)


Razor Views


MVC architecture



🔹 STEP 1: Create Project in Visual Studio


Open Visual Studio


Click Create a new project


Select ASP.NET Core Web App (Model-View-Controller)


Click Next


Project name:
Form1065Extractor



Framework:
.NET 6 or .NET 7



Authentication: None


Check ✔️ Use controllers with views


Click Create



🔹 STEP 2: Folder Structure (IMPORTANT)
Your project will look like this 👇
Form1065Extractor
│
├── Controllers
│   └── Form1065Controller.cs
│
├── Models
│   ├── Form1065Data.cs
│   └── ApplicationDbContext.cs
│
├── Services
│   └── AzureFormRecognizerService.cs
│
├── Views
│   ├── Form1065
│   │   ├── Index.cshtml
│   │   └── Result.cshtml
│   └── Shared
│       └── _Layout.cshtml
│
├── wwwroot
│
├── appsettings.json
└── Program.cs

Now I’ll explain each folder and the exact code to write.

🔹 MODELS (Database + Data Structure)
📁 Models/Form1065Data.cs
using System;

namespace Form1065Extractor.Models
{
    public class Form1065Data
    {
        public int Id { get; set; }

        public string PartnershipName { get; set; }
        public string EIN { get; set; }
        public string Address { get; set; }
        public int TaxYear { get; set; }

        public decimal TotalAssets { get; set; }
        public decimal GrossReceipts { get; set; }
        public decimal TotalIncome { get; set; }
        public decimal TotalDeductions { get; set; }
        public decimal OrdinaryBusinessIncome { get; set; }

        public DateTime CreatedAt { get; set; } = DateTime.Now;
    }
}


📁 Models/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;

namespace Form1065Extractor.Models
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions options) : base(options) { }

        public DbSet<Form1065Data> Form1065Datas { get; set; }
    }
}


🔹 SERVICES (Azure Form Recognizer Logic)
📁 Services/AzureFormRecognizerService.cs
using Azure;
using Azure.AI.FormRecognizer.DocumentAnalysis;
using Form1065Extractor.Models;

namespace Form1065Extractor.Services
{
    public class AzureFormRecognizerService
    {
        private readonly DocumentAnalysisClient _client;

        public AzureFormRecognizerService(IConfiguration config)
        {
            _client = new DocumentAnalysisClient(
                new Uri(config["AzureFormRecognizer:Endpoint"]),
                new AzureKeyCredential(config["AzureFormRecognizer:Key"])
            );
        }

        public async Task<Form1065Data> ExtractDataAsync(Stream fileStream)
        {
            var operation = await _client.AnalyzeDocumentAsync(
                WaitUntil.Completed,
                "prebuilt-document",
                fileStream
            );

            var doc = operation.Value.Documents.First();

            return new Form1065Data
            {
                PartnershipName = doc.Fields["NameOfPartnership"]?.Content,
                EIN = doc.Fields["EIN"]?.Content,
                TotalAssets = decimal.Parse(doc.Fields["TotalAssets"]?.Content ?? "0"),
                TaxYear = 2022
            };
        }
    }
}


🔹 CONTROLLER (Brain of MVC)
📁 Controllers/Form1065Controller.cs
using Microsoft.AspNetCore.Mvc;
using Form1065Extractor.Models;
using Form1065Extractor.Services;

namespace Form1065Extractor.Controllers
{
    public class Form1065Controller : Controller
    {
        private readonly AzureFormRecognizerService _service;
        private readonly ApplicationDbContext _context;

        public Form1065Controller(AzureFormRecognizerService service, ApplicationDbContext context)
        {
            _service = service;
            _context = context;
        }

        public IActionResult Index()
        {
            return View();
        }

        [HttpPost]
        public async Task<IActionResult> Upload(IFormFile pdfFile)
        {
            if (pdfFile == null || !pdfFile.FileName.EndsWith(".pdf"))
            {
                ViewBag.Error = "Only PDF files allowed";
                return View("Index");
            }

            using var stream = pdfFile.OpenReadStream();
            var data = await _service.ExtractDataAsync(stream);

            _context.Form1065Datas.Add(data);
            await _context.SaveChangesAsync();

            return View("Result", data);
        }
    }
}


🔹 VIEWS (UI)
📁 Views/Form1065/Index.cshtml
@{
    ViewData["Title"] = "Upload Form 1065";
}

<h2>Upload Form 1065 PDF</h2>

<form asp-action="Upload" method="post" enctype="multipart/form-data">
    <input type="file" name="pdfFile" required />
    <br /><br />
    <button type="submit">Upload & Extract</button>
</form>

@if (ViewBag.Error != null)
{
    <p style="color:red">@ViewBag.Error</p>
}


📁 Views/Form1065/Result.cshtml
@model Form1065Extractor.Models.Form1065Data

<h2>Extracted Data</h2>

<table border="1">
    <tr><td>Name</td><td>@Model.PartnershipName</td></tr>
    <tr><td>EIN</td><td>@Model.EIN</td></tr>
    <tr><td>Total Assets</td><td>@Model.TotalAssets</td></tr>
    <tr><td>Tax Year</td><td>@Model.TaxYear</td></tr>
</table>


🔹 CONFIGURATION
📁 appsettings.json
"AzureFormRecognizer": {
  "Endpoint": "YOUR_ENDPOINT",
  "Key": "YOUR_KEY"
},
"ConnectionStrings": {
  "DefaultConnection": "Server=.;Database=Form1065DB;Trusted_Connection=True;"
}


🔹 Program.cs
builder.Services.AddControllersWithViews();
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddScoped<AzureFormRecognizerService>();


🔹 WHAT YOU HAVE NOW ✅
✔ Proper MVC architecture
✔ Azure Form Recognizer integration
✔ SQL Database connection
✔ PDF upload validation
✔ Team-1 Form 1065 extraction
✔ Clean folder separation

🔹 NEXT STEPS (Tell me what you want)
I can now:
1️⃣ Add Edit / Update / Delete grid
2️⃣ Create JSON validation
3️⃣ Add History tracking
4️⃣ Create SQL scripts
5️⃣ Map exact Form-1065 fields
Just say what you want next and I’ll continue 🚀



@model Form1065Extractor.Models.Form1065Team1Data

<h2>Extracted Data (Team 1)</h2>

<table border="1" cellpadding="8">
    <tr>
        <td><strong>Name of Partnership</strong></td>
        <td>@Model.PartnershipName</td>
    </tr>
    <tr>
        <td><strong>EIN</strong></td>
        <td>@Model.EIN</td>
    </tr>
    <tr>
        <td><strong>Street Address</strong></td>
        <td>@Model.StreetAddress</td>
    </tr>
    <tr>
        <td><strong>City / State / ZIP</strong></td>
        <td>@Model.CityStateZip</td>
    </tr>
    <tr>
        <td><strong>Business Code Number</strong></td>
        <td>@Model.BusinessCodeNumber</td>
    </tr>
    <tr>
        <td><strong>Total Assets</strong></td>
        <td>@Model.TotalAssets</td>
    </tr>
    <tr>
        <td><strong>Applicable Boxes</strong></td>
        <td>@Model.ApplicableBoxes</td>
    </tr>
    <tr>
        <td><strong>Tax Year</strong></td>
        <td>@Model.TaxYear</td>
    </tr>
</table>

