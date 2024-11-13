---
title: "SuiteScript Development အတွက် IDE များ၊ Tools များနှင့် Development Env Setup လမ်းညွှန်"
date: 2024-11-13
author: "Tech Expert"
categories:
  - NetSuite
  - Development Tools
  - SuiteScript
tags:
  - IDE
  - Development Tools
  - JavaScript
  - TypeScript
  - Testing
  - Node.js
toc: true
excerpt: "SuiteScript Development အတွက် လိုအပ်သော IDE များ၊ Development Tools များ၊ Testing Frameworks များနှင့် Environment Setup အကြောင်း အသေးစိတ်လေ့လာခြင်း"
---

# SuiteScript Development Tools နှင့် Environment Setup လမ်းညွှန်

## မာတိကာ

1. [IDE ရွေးချယ်ခြင်း](#ide-ရွေးချယ်ခြင်း)
2. [Required Plugins များ](#required-plugins-များ)
3. [JavaScript Version အသုံးပြုခြင်း](#javascript-version-အသုံးပြုခြင်း)
4. [TypeScript Setup](#typescript-setup)
5. [Node.js Environment](#nodejs-environment)
6. [Testing Frameworks](#testing-frameworks)
7. [Development Workflow](#development-workflow)

## IDE ရွေးချယ်ခြင်း

### ၁။ Visual Studio Code

VS Code သည် SuiteScript development အတွက် အကောင်းဆုံး IDE တစ်ခုဖြစ်ပါသည်။

#### အားသာချက်များ

- Free and open-source
- Light-weight
- Rich extension ecosystem
- Built-in Git integration
- Integrated terminal
- IntelliSense support

#### Setup Instructions

1. VS Code ကို download နှင့် install လုပ်ခြင်း
2. SuiteScript-specific extensions များ install လုပ်ခြင်း
3. Workspace settings configure လုပ်ခြင်း

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "javascript.validate.enable": true,
  "typescript.validate.enable": true,
  "files.exclude": {
    "**/.git": true,
    "**/.DS_Store": true,
    "**/node_modules": true,
    "**/.deployment": true
  }
}
```

### ၂။ JetBrains WebStorm

Professional developers များအတွက် ပိုမိုသင့်လျော်သော IDE တစ်ခုဖြစ်ပါသည်။

#### အားသာချက်များ

- Advanced code analysis
- Built-in debugging tools
- Powerful refactoring capabilities
- Integrated development tools

## Required Plugins များ

### ၁။ VS Code Extensions

```javascript
// recommended-extensions.json
{
    "recommendations": [
        "oracle-cloud-infrastructure.netsuite-suitecloud-sdk",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "christian-kohler.path-intellisense",
        "streetsidesoftware.code-spell-checker",
        "eamodio.gitlens",
        "ms-vsliveshare.vsliveshare"
    ]
}
```

### ၂။ Essential Plugins Configuration

```javascript
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
  parser: "@typescript-eslint/parser",
  parserOptions: {
    ecmaVersion: 12,
    sourceType: "module",
  },
  plugins: ["@typescript-eslint"],
  rules: {
    "no-unused-vars": "warn",
    "no-console": "warn",
    "@typescript-eslint/explicit-module-boundary-types": "off",
  },
};
```

## JavaScript Version အသုံးပြုခြင်း

SuiteScript 2.x သည် ECMAScript 2015 (ES6) features အများစုကို support လုပ်ပါသည်။

### ၁။ Supported Features

```javascript
// Modern JavaScript features in SuiteScript
define(["N/record"], (record) => {
  // Arrow functions
  const createRecord = (type, data) => {
    // Template literals
    const recordId = `CUST_${Date.now()}`;

    // Destructuring
    const { name, email } = data;

    // Promise-based operations
    return Promise.resolve().then(() => {
      // Async operations
      const newRecord = record.create({
        type: record.Type.CUSTOMER,
        isDynamic: true,
      });

      // Spread operator
      const fields = {
        ...data,
        recordId,
      };

      // Object method shorthand
      return {
        save() {
          return newRecord.save();
        },
        fields,
      };
    });
  };

  return { createRecord };
});
```

### ၂။ Babel Configuration

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          node: "12",
        },
      },
    ],
    "@babel/preset-typescript",
  ],
  plugins: [
    "@babel/plugin-proposal-class-properties",
    "@babel/plugin-proposal-object-rest-spread",
  ],
};
```

## TypeScript Setup

### ၁။ TypeScript Configuration

```typescript
// tsconfig.json
{
    "compilerOptions": {
        "target": "es2015",
        "module": "amd",
        "lib": ["es2015", "dom"],
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "outDir": "./dist",
        "rootDir": "./src",
        "baseUrl": "./",
        "paths": {
            "N/*": ["node_modules/@types/netsuite-sdf/N/*"]
        }
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

### ၂။ TypeScript Example

```typescript
// src/CustomerService.ts
import { EntryPoints } from "@hitc/netsuite-types";
import * as record from "N/record";
import * as search from "N/search";

interface CustomerData {
  name: string;
  email: string;
  subsidiary: number;
}

export class CustomerService {
  public static createCustomer(data: CustomerData): number {
    const customer = record.create({
      type: record.Type.CUSTOMER,
      isDynamic: true,
    });

    customer.setValue({
      fieldId: "companyname",
      value: data.name,
    });

    customer.setValue({
      fieldId: "email",
      value: data.email,
    });

    customer.setValue({
      fieldId: "subsidiary",
      value: data.subsidiary,
    });

    return customer.save();
  }

  public static getCustomerDetails(customerId: number): CustomerData | null {
    const customerSearch = search.create({
      type: search.Type.CUSTOMER,
      filters: [["internalid", "is", customerId]],
      columns: ["companyname", "email", "subsidiary"],
    });

    const result = customerSearch.run().getRange({ start: 0, end: 1 })[0];

    if (!result) return null;

    return {
      name: result.getValue("companyname") as string,
      email: result.getValue("email") as string,
      subsidiary: result.getValue("subsidiary") as number,
    };
  }
}
```

## Node.js Environment

### ၁။ Project Setup

```json
// package.json
{
  "name": "suitescript-project",
  "version": "1.0.0",
  "description": "SuiteScript Development Project",
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "lint": "eslint . --ext .ts",
    "deploy": "suitecloud project:deploy",
    "validate": "suitecloud project:validate"
  },
  "dependencies": {
    "@hitc/netsuite-types": "^2021.2.4",
    "@oracle/suitecloud-unit-testing": "^1.2.1"
  },
  "devDependencies": {
    "@babel/core": "^7.15.0",
    "@babel/preset-env": "^7.15.0",
    "@babel/preset-typescript": "^7.15.0",
    "@types/jest": "^27.0.1",
    "@typescript-eslint/eslint-plugin": "^4.29.3",
    "@typescript-eslint/parser": "^4.29.3",
    "eslint": "^7.32.0",
    "jest": "^27.0.6",
    "prettier": "^2.3.2",
    "ts-jest": "^27.0.5",
    "typescript": "^4.3.5"
  }
}
```

### ၂။ Development Scripts

```bash
# Install dependencies
npm install

# Run TypeScript compiler
npm run build

# Run tests
npm run test

# Run linter
npm run lint

# Deploy to NetSuite
npm run deploy
```

## Testing Frameworks

### ၁။ Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: "ts-jest",
  testEnvironment: "node",
  moduleNameMapper: {
    "^N/(.*)$": "<rootDir>/node_modules/@hitc/netsuite-types/N/$1",
  },
  setupFiles: ["<rootDir>/jest.setup.js"],
  testMatch: ["**/__tests__/**/*.ts", "**/?(*.)+(spec|test).ts"],
  collectCoverage: true,
  coverageDirectory: "coverage",
  coverageReporters: ["text", "lcov"],
  collectCoverageFrom: ["src/**/*.ts", "!src/**/*.d.ts"],
};
```

### ၂။ Test Example

```typescript
// __tests__/CustomerService.test.ts
import { CustomerService } from "../src/CustomerService";
import * as record from "N/record";
import * as search from "N/search";

jest.mock("N/record");
jest.mock("N/search");

describe("CustomerService", () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe("createCustomer", () => {
    it("should create a customer record", () => {
      const mockSave = jest.fn().mockReturnValue(123);
      const mockSetValue = jest.fn();

      (record.create as jest.Mock).mockReturnValue({
        setValue: mockSetValue,
        save: mockSave,
      });

      const customerData = {
        name: "Test Company",
        email: "test@example.com",
        subsidiary: 1,
      };

      const customerId = CustomerService.createCustomer(customerData);

      expect(customerId).toBe(123);
      expect(record.create).toHaveBeenCalledWith({
        type: record.Type.CUSTOMER,
        isDynamic: true,
      });
      expect(mockSetValue).toHaveBeenCalledTimes(3);
    });
  });

  describe("getCustomerDetails", () => {
    it("should return customer details", () => {
      const mockResult = {
        getValue: jest.fn().mockImplementation((field) => {
          const values = {
            companyname: "Test Company",
            email: "test@example.com",
            subsidiary: 1,
          };
          return values[field];
        }),
      };

      const mockRun = jest.fn().mockReturnValue({
        getRange: jest.fn().mockReturnValue([mockResult]),
      });

      (search.create as jest.Mock).mockReturnValue({
        run: mockRun,
      });

      const result = CustomerService.getCustomerDetails(123);

      expect(result).toEqual({
        name: "Test Company",
        email: "test@example.com",
        subsidiary: 1,
      });
    });
  });
});
```

## Development Workflow

### ၁။ Git Workflow

```bash
# .gitignore
node_modules/
dist/
coverage/
.env
*.log
.DS_Store
.deployment/
```

### ၂။ CI/CD Pipeline

```yaml
# .github/workflows/netsuite-deployment.yml
name: NetSuite Deployment

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: "14"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Lint
        run: npm run lint

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Deploy to NetSuite
        run: npm run deploy
        env:
          SUITECLOUD_EMAIL: ${{ secrets.NETSUITE_EMAIL }}
          SUITECLOUD_PASSWORD: ${{ secrets.NETSUITE_PASSWORD }}
          SUITECLOUD_ACCOUNT: ${{ secrets.NETSUITE_ACCOUNT }}
```

## နိဂုံး

SuiteScript development အတွက် မှန်ကန်သော tools များနှင့် environment setup သည် development process ကို ပိုမိုထိရောက်စေပါသည်။ TypeScript အသုံးပြုခြင်းဖြင့် type safety ရရှိပြီး၊ automated testing များဖြင့် code quality ကို ထိန်းသိမ်းနိုင်ပါသည်။ ကောင်းမွန်သော development workflow တစ်ခုသည် team collaboration နှင့် code maintenance ကို ပိုမိုလွယ်ကူစေပါသည်။

Development tools များကို project requirements နှင့် team preferences ပေါ်မူတည်၍ ရွေးချယ်အသုံးပြုသင့်ပါသည်။
