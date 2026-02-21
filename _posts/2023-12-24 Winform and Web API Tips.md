# C# WinForm and Web API
<!-- TOC -->

- [C# WinForm and Web API](#c-winform-and-web-api)
    - [Automapper](#automapper)
        - [List and Array](#list-and-array)
        - [Web API Initialization](#web-api-initialization)
    - [DataAnnotations Validation Attributes in Windows Forms](#dataannotations-validation-attributes-in-windows-forms)
        - [Model Attributes StringLength, DisplayName, Display](#model-attributes-stringlength-displayname-display)
        - [ErrorProvider for auto validation](#errorprovider-for-auto-validation)
    - [Use the StringLength attribute to setup TextBox MaxLength property in the form](#use-the-stringlength-attribute-to-setup-textbox-maxlength-property-in-the-form)
    - [Database Key column is not auto increment.  Cause EF append record generate conflict](#database-key-column-is-not-auto-increment--cause-ef-append-record-generate-conflict)
        - [Using SQL to change column property](#using-sql-to-change-column-property)
        - [Entity Frame with Max function](#entity-frame-with-max-function)
    - [In Entity Framework, how to change the record's key value?](#in-entity-framework-how-to-change-the-records-key-value)

<!-- /TOC -->
         {
            dateTimePicker.Checked = false;
         } else {
            dateTimePicker.Checked = true;
            try {
               dateTimePicker.Value = (DateTime)value;
            }
            catch {
               dateTimePicker.Value = dateTimePicker.MinDate;
            }
         }
      }
      private DateTime? GetDateTimePicker(DateTimePicker dateTimePicker)
      {
         if (!dateTimePicker.Checked)
            return null;
         return dateTimePicker.Value;
      }
```   

  * Load value when form loaded.

```c#
      private void EmployeeEditDialog_Load(object sender, EventArgs e)
      {
         employeeViewModelBindingSource.DataSource = mValue;

         SetDateTimePicker(dateTimePicker1, mValue.DDateHired);
         SetDateTimePicker(dateTimePicker2, mValue.DBirthday);
         SetDateTimePicker(dateTimePicker3, mValue.DTerminated);
         SetDateTimePicker(dateTimePicker4, mValue.DDateEntered);
         SetDateTimePicker(dateTimePicker5, mValue.DDateModified);

      }
``

   * Collect value when Ok button clicked.

```c#
      private void button1_Click(object sender, EventArgs e)
      {
         mValue.DDateHired = GetDateTimePicker(dateTimePicker1);
         mValue.DBirthday = GetDateTimePicker(dateTimePicker2);
         mValue.DTerminated = GetDateTimePicker(dateTimePicker3);
         mValue.DDateEntered = GetDateTimePicker(dateTimePicker4);
         mValue.DDateModified = GetDateTimePicker(dateTimePicker5);
         DialogResult = DialogResult.OK;
      }
```

## Automapper

### [List and Array](https://docs.automapper.org/en/stable/Lists-and-arrays.html)

### Web API Initialization

In program.cs, before add dbContext

```c#
builder.Services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
// Add DbContext
builder.Services.AddDbContext<ForexdbContext>(options =>
   options.UseSqlServer(configuration.GetConnectionString("FOREXDB")));

var app = builder.Build();
```

## DataAnnotations Validation Attributes in Windows Forms

* [DataAnnotations Validation Attributes in Windows Forms](https://www.reza-aghaei.com/dataannotations-validation-attributes-in-windows-forms/)

### Model Attributes( StringLength, DisplayName, Display)

```c#
namespace MemberService.Models
{
   public class EmployeeViewModel
   {
      [DisplayName("Employee ID")]
      public int IEmployeeId { get; set; }

      [StringLength(35, MinimumLength = 2, ErrorMessage = "The {0} value cannot exceed {1} characters and at least {2} characters.")]
      [DisplayName("Full Name")]
      public string? SFullName { get; set; }

      [DisplayName("Current Employed")]

   }
}
```

The error message for Field `SFullName` will be

"The SFullName value cannot exceed 35 characters and at least 2 characters."

* Problem: Can we use "Display Name" of field instead, not the database field name.  Like 
"The Full Name value cannot exceed 35 characters and at least 2 characters."

  * Use `Display` instead of `DisplayName` attribute

```c#
      [Display(Name="Full Name")]
      [StringLength(35, MinimumLength = 2, ErrorMessage = "The {0} value cannot exceed {1} characters and at least {2} characters.")]
      public string? SFullName { get; set; }
```

### ErrorProvider for auto validation

In order to be able to do validation with ErrorProvider, the model need to implement IDataErrorInfo interface.

```c#
   public class EmployeeEditModel: EmployeeViewModel, IDataErrorInfo
   {
      [Browsable(false)]
      public string this[string property]
      {
         get
         {
            var propertyDescriptor = TypeDescriptor.GetProperties(this)[property];
            if (propertyDescriptor == null)
               return string.Empty;

            var results = new List<ValidationResult>();
            var result = Validator.TryValidateProperty(
                                      propertyDescriptor.GetValue(this),
                                      new ValidationContext(this, null, null)
                                      { MemberName = property },
                                      results);
            if (!result)
               return results.First().ErrorMessage;
            return string.Empty;
         }
      }

      [Browsable(false)]
      public string Error
      {
         get
         {
            var results = new List<ValidationResult>();
            var result = Validator.TryValidateObject(this,
                new ValidationContext(this, null, null), results, true);
            if (!result)
               return string.Join("\n", results.Select(x => x.ErrorMessage));
            else
               return null;
         }
      }

   }

```

## Use the StringLength attribute to setup TextBox `MaxLength` property in the form

```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Reflection;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace UserManagement.Utils
{
   public static class TexBoxUtils
   {
      public static void SetMaxLengthFromStringLengthAttributes(Control container)
      {
         foreach (Control control in container.Controls)
         {
            if (control is TextBox textBox)
            {
               ApplyStringLengthAttribute(textBox);
            }
            else if (control.HasChildren)
            {
               SetMaxLengthFromStringLengthAttributes(control);
            }
         }
      }

      private static void ApplyStringLengthAttribute(TextBox textBox)
      {
         PropertyDescriptor property = GetBindingProperty(textBox);

         if (property != null)
         {
            StringLengthAttribute stringLengthAttribute = property
                .Attributes
                .OfType<StringLengthAttribute>()
                .FirstOrDefault();

            if (stringLengthAttribute != null)
            {
               textBox.MaxLength = stringLengthAttribute.MaximumLength;
            }
         }
      }
      private static PropertyDescriptor GetBindingProperty(TextBox textBox)
      {
         Binding binding = textBox.DataBindings["Text"];

         if (binding != null)
         {
            CurrencyManager currencyManager = binding.BindingManagerBase as CurrencyManager;

            if (currencyManager != null)
            {
               // Get the underlying data source
               object dataSource = currencyManager.List[0];

               // Get the property associated with the TextBox
               string propertyName = binding.BindingMemberInfo.BindingField;

               // Get the property descriptors
               PropertyDescriptorCollection propertyDescriptors = TypeDescriptor.GetProperties(dataSource);

               // Find the PropertyDescriptor by property name
               PropertyDescriptor propertyDescriptor = propertyDescriptors.Find(propertyName, true);

               return propertyDescriptor;
            }
         }
         return null;
      }

   }
}

```

## Database Key column is not auto increment.  Cause EF append record generate conflict

- Solutions

  1. Use SQL to change column property
  2. Manually key column assign value in the record
  3. Auto increment with Max in EF

### Using SQL to change column property

1. Add a New Auto-Increment Column:
Add a new column with the IDENTITY property to the table. This will be your new auto-increment column.

```SQL
ALTER TABLE YourTableName
ADD NewAutoIncrementColumn INT IDENTITY(1,1);
```

2. Update the New Column with Existing Values:
Update the new column with the existing values from the original column.

```sql
UPDATE YourTableName
SET NewAutoIncrementColumn = OriginalColumn;
```

3. Drop the Original Column:

```sql
ALTER TABLE YourTableName
DROP COLUMN OriginalColumn;
```

4. Rename the New Column:
Rename the new auto-increment column to the original column name.

```sql

EXEC sp_rename 'YourTableName.NewAutoIncrementColumn', 'OriginalColumn', 'COLUMN';
```

This process effectively replaces the existing column with a new auto-increment column. However, keep in mind that this approach may not preserve foreign key relationships or other constraints related to the original column. Before making such changes, ensure that you have a backup of your database and thoroughly test the process in a safe environment.

### Entity Frame with Max function

* Using DbContext.Tables.MaxAsync to calculate next available incrmented value

```C#
// POST api/<EmployeeController>
      [HttpPost]
      public async Task<ActionResult<BrokerViewModel>> PostBroker([FromBody] BrokerViewModel model)
      {
         if (_context.Brokers == null)
         {
            return Problem("Entity set 'ForexdbContext.Brokers'  is null.");
         }
         
         var broker =  _mapper.Map<Broker>(model);
         int v = await _context.Brokers.MaxAsync(p => ((int?) p.IBrokerId) ?? 0)+1;
         broker.IBrokerId = v;
         _context.Brokers.Add(broker);
         await _context.SaveChangesAsync();         
         return CreatedAtAction("PostBroker", new { id = broker.IBrokerId }, _mapper.Map<BrokerViewModel>( broker));
      }

```

## In Entity Framework, how to change the record's key value?

* Can't update the record directly.  Need to delete and add new one

```c#
      [HttpPut("{id}")]
      public async Task<IActionResult> PutBroker(int id, BrokerViewModel model)
      {
         if (id != model.IBrokerId)
         {
            //  return BadRequest();
            // add new one
            var broker1 = _mapper.Map<Broker>(model);
            _context.Brokers.Add(broker1);
            await _context.SaveChangesAsync();

            // delete original and add new one
            var broker0 = await _context.Brokers.Where(b => b.IBrokerId == id).FirstOrDefaultAsync<Broker>();
            if (broker0 == null)
            {
               return NotFound();
            }
            _context.Brokers.Remove(broker0);
            await _context.SaveChangesAsync();
            return NoContent();
         }

         ...

```
