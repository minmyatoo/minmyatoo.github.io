---
title: "SuiteScript Development - အခြေခံမှ အဆင့်မြင့်နည်းပညာများအထိ"
date: 2024-11-13
author: "Tech Expert"
categories:
  - NetSuite
  - SuiteScript
  - Development
tags:
  - SuiteScript
  - JavaScript
  - NetSuite Development
  - API
  - Programming
toc: true
excerpt: "SuiteScript 2.0 နည်းပညာအား အခြေခံမှစ၍ အဆင့်မြင့်နည်းပညာများအထိ အသေးစိတ်လေ့လာခြင်း"
---
# SuiteScript Development အခြေခံမှ အဆင့်မြင့်အထိ

## မာတိကာ

1. [SuiteScript ဆိုတာဘာလဲ](#suitescript-ဆိုတာဘာလဲ)
2. [Script Types များ](#script-types-များ)
3. [Development Environment Setup](#development-environment-setup)
4. [အခြေခံ APIs များ](#အခြေခံ-apis-များ)
5. [Advanced Features များ](#advanced-features-များ)
6. [Best Practices များ](#best-practices-များ)
7. [Real-world Examples](#real-world-examples)

## SuiteScript ဆိုတာဘာလဲ

SuiteScript သည် NetSuite platform အတွက် JavaScript-based programming language တစ်ခုဖြစ်ပါသည်။ ၎င်းသည် NetSuite ၏ functionality များကို customize လုပ်ရန်နှင့် extend လုပ်ရန်အတွက် အသုံးပြုပါသည်။

### SuiteScript 2.0 ၏ အားသာချက်များ

1. **Modern JavaScript Features**

   - ES6+ features များ support လုပ်ခြင်း
   - Modules system ပါဝင်ခြင်း
   - Promise-based operations များ
2. **Type Safety**

   - JSDoc annotations
   - Type checking
   - Code completion support
3. **Performance Improvements**

   - Better memory management
   - Optimized execution
   - Improved error handling

## Script Types များ

### ၁။ Client Script

Client-side မှာ run မယ့် script တွေအတွက် အသုံးပြုပါတယ်။

```javascript
/**
 * @NApiVersion 2.x
 * @NScriptType ClientScript
 */
define(["N/currentRecord", "N/ui/message"], function (currentRecord, message) {
  // Page initialization
  function pageInit(context) {
    try {
      var currentRec = context.currentRecord;
      var recordType = currentRec.type;

      // Show welcome message
      message
        .create({
          title: "Welcome",
          message: "Record Type: " + recordType,
          type: message.Type.INFORMATION,
        })
        .show({
          duration: 5000, // 5 seconds
        });
    } catch (e) {
      console.error("Error in pageInit", e);
    }
  }

  // Field changed event
  function fieldChanged(context) {
    try {
      var currentRec = context.currentRecord;
      var fieldId = context.fieldId;

      if (fieldId === "custpage_amount") {
        var amount = currentRec.getValue({
          fieldId: "custpage_amount",
        });

        // Calculate tax (10%)
        var tax = amount * 0.1;

        currentRec.setValue({
          fieldId: "custpage_tax",
          value: tax,
        });
      }
    } catch (e) {
      console.error("Error in fieldChanged", e);
    }
  }

  return {
    pageInit: pageInit,
    fieldChanged: fieldChanged,
  };
});
```

### ၂။ User Event Script

Record တစ်ခု create, edit, delete လုပ်တဲ့အချိန်မှာ trigger ဖြစ်မယ့် script တွေပါ။

```javascript
/**
 * @NApiVersion 2.x
 * @NScriptType UserEventScript
 */
define(["N/record", "N/email", "N/runtime"], function (record, email, runtime) {
  function beforeSubmit(context) {
    try {
      if (context.type !== context.UserEventType.CREATE) return;

      var newRecord = context.newRecord;

      // Add current timestamp
      var now = new Date();
      newRecord.setValue({
        fieldId: "custbody_created_timestamp",
        value: now,
      });

      // Add current user
      var user = runtime.getCurrentUser();
      newRecord.setValue({
        fieldId: "custbody_created_by",
        value: user.id,
      });
    } catch (e) {
      log.error("Error in beforeSubmit", e);
    }
  }

  function afterSubmit(context) {
    try {
      if (context.type !== context.UserEventType.CREATE) return;

      var newRecord = context.newRecord;
      var recordId = newRecord.id;

      // Send notification email
      email.send({
        author: runtime.getCurrentUser().id,
        recipients: "admin@company.com",
        subject: "New Record Created",
        body:
          "Record ID: " +
          recordId +
          "\nType: " +
          newRecord.type +
          "\nDate: " +
          new Date(),
      });
    } catch (e) {
      log.error("Error in afterSubmit", e);
    }
  }

  return {
    beforeSubmit: beforeSubmit,
    afterSubmit: afterSubmit,
  };
});
```

### ၃။ Scheduled Script

Background မှာ run မယ့် batch processes တွေအတွက် အသုံးပြုပါတယ်။

```javascript
/**
 * @NApiVersion 2.x
 * @NScriptType ScheduledScript
 */
define(["N/search", "N/record", "N/email"], function (search, record, email) {
  function execute(context) {
    try {
      // Search for inactive customers
      var customerSearch = search.create({
        type: search.Type.CUSTOMER,
        filters: [
          ["isinactive", "is", "F"],
          "AND",
          ["lastorderdate", "before", "lastmonth"],
        ],
        columns: ["entityid", "email", "lastorderdate"],
      });

      var inactiveCustomers = [];

      customerSearch.run().each(function (result) {
        inactiveCustomers.push({
          id: result.id,
          name: result.getValue("entityid"),
          email: result.getValue("email"),
          lastOrder: result.getValue("lastorderdate"),
        });
        return true;
      });

      // Send report
      if (inactiveCustomers.length > 0) {
        email.send({
          author: -5,
          recipients: "sales@company.com",
          subject: "Inactive Customers Report",
          body: JSON.stringify(inactiveCustomers, null, 2),
        });

        log.audit({
          title: "Inactive Customers Report",
          details: inactiveCustomers.length + " customers found",
        });
      }
    } catch (e) {
      log.error("Error in execute", e);
    }
  }

  return {
    execute: execute,
  };
});
```

## Development Environment Setup

### ၁။ Required Tools

- SuiteCloud IDE
- Node.js
- VS Code with NetSuite extensions
- SuiteCloud Development Framework (SDF)

### ၂။ Environment Configuration

```javascript
// suitecloud.config.js
module.exports = {
  defaultProjectFolder: "src",
  commands: {
    "project:create": {
      // Project creation settings
      parentDirectory: ".",
      overwrite: true,
      projectName: "my-suitescript-project",
      projectVersion: "1.0.0",
      publisher: "My Company",
    },
    "file:upload": {
      // File upload settings
      paths: ["src/**/*.js"],
      excludeProperties: false,
      excludeFiles: false,
    },
  },
};
```

## အခြေခံ APIs များ

### ၁။ Record API

```javascript
// Record loading and manipulation
require(["N/record"], function (record) {
  // Create new record
  var salesOrder = record.create({
    type: record.Type.SALES_ORDER,
    isDynamic: true,
  });

  // Set field values
  salesOrder.setValue({
    fieldId: "entity",
    value: 123, // Customer ID
  });

  // Add line item
  salesOrder.selectNewLine({
    sublistId: "item",
  });

  salesOrder.setCurrentSublistValue({
    sublistId: "item",
    fieldId: "item",
    value: 456, // Item ID
  });

  salesOrder.setCurrentSublistValue({
    sublistId: "item",
    fieldId: "quantity",
    value: 1,
  });

  salesOrder.commitLine({
    sublistId: "item",
  });

  // Save record
  var recordId = salesOrder.save();
});
```

### ၂။ Search API

```javascript
// Advanced search functionality
require(["N/search"], function (search) {
  // Create search
  var invoiceSearch = search.create({
    type: search.Type.INVOICE,
    filters: [
      ["mainline", "is", "T"],
      "AND",
      ["status", "anyof", "CustInvc:A"],
    ],
    columns: [
      search.createColumn({ name: "tranid" }),
      search.createColumn({ name: "amount" }),
      search.createColumn({
        name: "daysopen",
        sort: search.Sort.DESC,
      }),
    ],
  });

  // Run search and process results
  invoiceSearch.run().each(function (result) {
    log.debug({
      title: "Invoice",
      details: {
        id: result.id,
        number: result.getValue("tranid"),
        amount: result.getValue("amount"),
        daysOpen: result.getValue("daysopen"),
      },
    });
    return true;
  });
});
```

## Advanced Features များ

### ၁။ Custom Modules

```javascript
/**
 * @NApiVersion 2.x
 * @NModuleScope Public
 */
define(["N/format"], function (format) {
  function formatCurrency(amount, currency) {
    if (!amount) return "0.00";

    return format.format({
      value: amount,
      type: format.Type.CURRENCY,
      currency: currency || "USD",
    });
  }

  function calculateTax(amount, rate) {
    if (!amount || !rate) return 0;

    return parseFloat(amount) * (parseFloat(rate) / 100);
  }

  return {
    formatCurrency: formatCurrency,
    calculateTax: calculateTax,
  };
});
```

### ၂။ Map/Reduce Script

```javascript
/**
 * @NApiVersion 2.x
 * @NScriptType MapReduceScript
 */
define(["N/record", "N/search"], function (record, search) {
  function getInputData() {
    return search.create({
      type: search.Type.SALES_ORDER,
      filters: [
        ["mainline", "is", "T"],
        "AND",
        ["status", "anyof", "SalesOrd:A"],
      ],
      columns: ["tranid", "amount", "entity"],
    });
  }

  function map(context) {
    var searchResult = JSON.parse(context.value);
    var customerId = searchResult.values.entity.value;

    context.write({
      key: customerId,
      value: {
        orderId: searchResult.id,
        amount: searchResult.values.amount,
      },
    });
  }

  function reduce(context) {
    var customerId = context.key;
    var orders = context.values.map(function (value) {
      return JSON.parse(value);
    });

    var totalAmount = 0;
    orders.forEach(function (order) {
      totalAmount += parseFloat(order.amount);
    });

    // Update customer record
    record.submitFields({
      type: record.Type.CUSTOMER,
      id: customerId,
      values: {
        custentity_total_orders: orders.length,
        custentity_total_amount: totalAmount,
      },
    });
  }

  function summarize(summary) {
    log.audit({
      title: "Process Summary",
      details: "Total processed: " + summary.reduceSummary.total,
    });
  }

  return {
    getInputData: getInputData,
    map: map,
    reduce: reduce,
    summarize: summarize,
  };
});
```

## Best Practices များ

### ၁။ Error Handling

```javascript
function safelyExecute(operation) {
  try {
    return operation();
  } catch (e) {
    log.error({
      title: "Operation Failed",
      details: e,
    });
    throw e;
  }
}

// Usage example
safelyExecute(function () {
  // Your code here
});
```

### ၂။ Performance Optimization

```javascript
// Use search.PagedData for large datasets
function processLargeDataset() {
  var pagedData = search
    .create({
      type: search.Type.SALES_ORDER,
      columns: ["internalid"],
    })
    .runPaged({
      pageSize: 1000,
    });

  pagedData.pageRanges.forEach(function (pageRange) {
    var page = pagedData.fetch({
      index: pageRange.index,
    });

    page.data.forEach(function (result) {
      // Process each result
    });
  });
}
```

## Real-world Examples

### ၁။ Customer Statement Generator

```javascript
/**
 * @NApiVersion 2.x
 * @NScriptType Suitelet
 */
define(["N/render", "N/search", "N/file"], function (render, search, file) {
  function onRequest(context) {
    if (context.request.method !== "GET") return;

    var customerId = context.request.parameters.custid;

    // Get customer transactions
    var transactions = getCustomerTransactions(customerId);

    // Generate PDF
    var pdfFile = generatePDF(transactions);

    // Send response
    context.response.writeFile({
      file: pdfFile,
      isInline: true,
    });
  }

  function getCustomerTransactions(customerId) {
    var transactionSearch = search.create({
      type: search.Type.TRANSACTION,
      filters: [
        ["mainline", "is", "T"],
        "AND",
        ["customer.internalid", "anyof", customerId],
      ],
      columns: ["trandate", "type", "tranid", "amount", "status"],
    });

    var transactions = [];
    transactionSearch.run().each(function (result) {
      transactions.push({
        date: result.getValue("trandate"),
        type: result.getValue("type"),
        number: result.getValue("tranid"),
        amount: result.getValue("amount"),
        status: result.getValue("status"),
      });
      return true;
    });

    return transactions;
  }

  function generatePDF(transactions) {
    var template = file.load({
      id: "custtemplate_customer_statement",
    });

    var renderer = render.create();
    renderer.templateContent = template.getContents();

    renderer.addCustomDataSource({
      format: render.DataSource.OBJECT,
      alias: "JSON",
      data: {
        transactions: transactions,
        generatedDate: new Date(),
      },
    });

    return renderer.renderAsPdf();
  }

  return {
    onRequest: onRequest,
  };
});
```

### ၂။ Inventory Reorder System

```javascript
/**
 * @NApiVersion 2.x
 * @NScriptType WorkflowActionScript
 */
define(["N/record", "N/search", "N/email"], function (record, search, email) {
  function onAction(context) {
    // Get reorder point and current quantity
    var item = record.load({
      type: record.Type.INVENTORY_ITEM,
      id: context.newRecord.id,
    });

    var reorderPoint = item.getValue("reorderpoint");
    var quantityOnHand = item.getValue("quantityonhand");

    if (quantityOnHand <= reorderPoint) {
      // Create purchase order
      var purchaseOrder = createPurchaseOrder(item);

      // Send notification
      sendReorderNotification(item, purchaseOrder);
    }
  }

  function createPurchaseOrder(item) {
    var po = record.create({
      type: record.Type.PURCHASE_ORDER,
      isDynamic: true,
    });

    // Set vendor from preferred vendor field
    po.setValue({
      fieldId: "entity",
      value: item.getValue("preferredvendor"),
    });

    // Add item to PO
    po.selectNewLine({
      sublistId: "item",
    });

    po.setCurrentSublistValue({
      sublistId: "item",
      fieldId: "item",
      value: item.id,
    });

    po.setCurrentSublistValue({
      sublistId: "item",
      fieldId: "quantity",
      value: item.getValue("reordermultiple"),
    });

    po.commitLine({
      sublistId: "item",
    });

    return po.save();
  }

  function sendReorderNotification(item, poId) {
    email.send({
      author: -5,
      recipients: "purchasing@company.com",
      subject: "Reorder Alert: " + item.getValue("itemid"),
      body:
        "Purchase Order " +
        poId +
        " has been created for " +
        item.getValue("itemid") +
        ".\n\n" +
        "Current Quantity: " +
        item.getValue("quantityonhand") +
        "\n" +
        "Reorder Point: " +
        item.getValue("reorderpoint"),
    });
  }

  return {
    onAction: onAction,
  };
});
```

## စွမ်းဆောင်ရည်မြှင့်တင်ခြင်းနှင့် Debugging

### ၁။ Performance Monitoring

```javascript
/**
 * @NApiVersion 2.x
 * @NModuleScope Public
 */
define(["N/log"], function (log) {
  var PerformanceMonitor = {
    startTime: null,

    start: function () {
      this.startTime = Date.now();
      log.debug({
        title: "Performance Monitor",
        details: "Started monitoring",
      });
    },

    end: function (operation) {
      var endTime = Date.now();
      var duration = endTime - this.startTime;

      log.audit({
        title: "Performance Results",
        details: operation + " took " + duration + "ms",
      });

      return duration;
    },
  };

  return PerformanceMonitor;
});
```

### ၂။ Debug Utilities

```javascript
/**
 * @NApiVersion 2.x
 * @NModuleScope Public
 */
define(["N/log"], function (log) {
  var Debugger = {
    verbose: false,

    setVerbose: function (value) {
      this.verbose = value;
    },

    log: function (title, details) {
      if (this.verbose) {
        log.debug({
          title: title,
          details:
            typeof details === "object"
              ? JSON.stringify(details, null, 2)
              : details,
        });
      }
    },

    logObject: function (obj, depth) {
      depth = depth || 0;
      var indent = "  ".repeat(depth);

      for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
          var value = obj[key];

          if (typeof value === "object" && value !== null) {
            log.debug({
              title: indent + key,
              details: "{",
            });
            this.logObject(value, depth + 1);
            log.debug({
              title: indent + "}",
              details: "",
            });
          } else {
            log.debug({
              title: indent + key,
              details: value,
            });
          }
        }
      }
    },
  };

  return Debugger;
});
```

## လုပ်ငန်းလုပ်ဆောင်မှုကို မှတ်တမ်းတင်ခြင်း (Logging)

```javascript
/**
 * @NApiVersion 2.x
 * @NScriptType UserEventScript
 */
define(["N/log", "./lib/Debugger"], function (log, Debugger) {
  function beforeSubmit(context) {
    Debugger.setVerbose(true);

    try {
      Debugger.log("Context", {
        type: context.type,
        recordType: context.newRecord.type,
      });

      // Process record...

      Debugger.log("Processing completed", "Success");
    } catch (e) {
      log.error({
        title: "Processing Error",
        details: e.message,
      });
      throw e;
    }
  }

  return {
    beforeSubmit: beforeSubmit,
  };
});
```

## နိဂုံး

SuiteScript သည် NetSuite platform ကို customize လုပ်ရန်အတွက် အလွန်အရေးပါသော tool တစ်ခုဖြစ်ပါသည်။ ကောင်းမွန်သော code architecture နှင့် best practices များကို လိုက်နာခြင်းဖြင့် ရေရှည်တည်တံ့ပြီး စွမ်းဆောင်ရည်မြင့်မားသော solutions များကို တည်ဆောက်နိုင်မှာ ဖြစ်ပါတယ်။

အထက်ပါ code examples များနှင့် patterns များကို နားလည်အောင်လေ့လာပြီး မိမိ၏ project များတွင် အသုံးချနိုင်ပါတယ်။ SuiteScript development သည် continuous learning process တစ်ခုဖြစ်တဲ့အတွက် NetSuite ရဲ့ documentation များနှင့် community resources များကို မှန်မှန်လေ့လာဖို့ လိုအပ်ပါတယ်။
