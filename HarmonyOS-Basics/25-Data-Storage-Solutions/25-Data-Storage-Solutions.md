# 25-Data Storage Solutions

## Introduction

Data storage is a critical aspect of HarmonyOS application development that determines how your app persists user data, preferences, and application state. This article explores various storage options available in HarmonyOS, including Preferences, RDB (Relational Database), KV (Key-Value) Store, and file system operations.

## Storage Options Overview

HarmonyOS provides several data storage mechanisms:

- **Preferences**: Simple key-value storage for app preferences
- **RDB**: SQLite-based relational database for complex data
- **KV Store**: Distributed key-value storage
- **File Storage**: Direct file system operations
- **DataShare**: Cross-application data sharing

## Preferences Storage

### Basic Preferences Operations

```typescript
import dataPreferences from "@ohos.data.preferences";

export class PreferencesManager {
  private static readonly STORE_NAME = "app_preferences";
  private static preferences: dataPreferences.Preferences | null = null;

  static async initialize(context: Context): Promise<void> {
    try {
      this.preferences = await dataPreferences.getPreferences(
        context,
        this.STORE_NAME
      );
    } catch (error) {
      console.error("Failed to initialize preferences:", error);
      throw error;
    }
  }

  static async setString(key: string, value: string): Promise<void> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      await this.preferences.put(key, value);
      await this.preferences.flush();
    } catch (error) {
      console.error(`Failed to set string preference ${key}:`, error);
      throw error;
    }
  }

  static async getString(
    key: string,
    defaultValue: string = ""
  ): Promise<string> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      return (await this.preferences.get(key, defaultValue)) as string;
    } catch (error) {
      console.error(`Failed to get string preference ${key}:`, error);
      return defaultValue;
    }
  }

  static async setNumber(key: string, value: number): Promise<void> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      await this.preferences.put(key, value);
      await this.preferences.flush();
    } catch (error) {
      console.error(`Failed to set number preference ${key}:`, error);
      throw error;
    }
  }

  static async getNumber(
    key: string,
    defaultValue: number = 0
  ): Promise<number> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      return (await this.preferences.get(key, defaultValue)) as number;
    } catch (error) {
      console.error(`Failed to get number preference ${key}:`, error);
      return defaultValue;
    }
  }

  static async setBoolean(key: string, value: boolean): Promise<void> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      await this.preferences.put(key, value);
      await this.preferences.flush();
    } catch (error) {
      console.error(`Failed to set boolean preference ${key}:`, error);
      throw error;
    }
  }

  static async getBoolean(
    key: string,
    defaultValue: boolean = false
  ): Promise<boolean> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      return (await this.preferences.get(key, defaultValue)) as boolean;
    } catch (error) {
      console.error(`Failed to get boolean preference ${key}:`, error);
      return defaultValue;
    }
  }

  static async setObject<T>(key: string, value: T): Promise<void> {
    try {
      const jsonString = JSON.stringify(value);
      await this.setString(key, jsonString);
    } catch (error) {
      console.error(`Failed to set object preference ${key}:`, error);
      throw error;
    }
  }

  static async getObject<T>(key: string, defaultValue: T): Promise<T> {
    try {
      const jsonString = await this.getString(key, "");
      if (jsonString) {
        return JSON.parse(jsonString) as T;
      }
      return defaultValue;
    } catch (error) {
      console.error(`Failed to get object preference ${key}:`, error);
      return defaultValue;
    }
  }

  static async remove(key: string): Promise<void> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      await this.preferences.delete(key);
      await this.preferences.flush();
    } catch (error) {
      console.error(`Failed to remove preference ${key}:`, error);
      throw error;
    }
  }

  static async clear(): Promise<void> {
    if (!this.preferences) {
      throw new Error("Preferences not initialized");
    }

    try {
      await this.preferences.clear();
      await this.preferences.flush();
    } catch (error) {
      console.error("Failed to clear preferences:", error);
      throw error;
    }
  }
}

// User preferences model
interface UserPreferences {
  theme: "light" | "dark";
  language: string;
  notifications: boolean;
  autoSave: boolean;
  fontSize: number;
}

// Application settings manager
export class AppSettingsManager {
  private static readonly USER_PREFS_KEY = "user_preferences";

  static async saveUserPreferences(
    preferences: UserPreferences
  ): Promise<void> {
    await PreferencesManager.setObject(this.USER_PREFS_KEY, preferences);
  }

  static async loadUserPreferences(): Promise<UserPreferences> {
    const defaultPrefs: UserPreferences = {
      theme: "light",
      language: "en",
      notifications: true,
      autoSave: true,
      fontSize: 16,
    };

    return await PreferencesManager.getObject(
      this.USER_PREFS_KEY,
      defaultPrefs
    );
  }

  static async updateTheme(theme: "light" | "dark"): Promise<void> {
    const prefs = await this.loadUserPreferences();
    prefs.theme = theme;
    await this.saveUserPreferences(prefs);
  }

  static async updateLanguage(language: string): Promise<void> {
    const prefs = await this.loadUserPreferences();
    prefs.language = language;
    await this.saveUserPreferences(prefs);
  }
}
```

## Relational Database (RDB)

### Database Setup and Operations

```typescript
import relationalStore from "@ohos.data.relationalStore";

// Database configuration
const DB_CONFIG: relationalStore.StoreConfig = {
  name: "AppDatabase.db",
  securityLevel: relationalStore.SecurityLevel.S1,
};

// Database table schemas
const CREATE_USERS_TABLE = `
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    age INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
  )
`;

const CREATE_POSTS_TABLE = `
  CREATE TABLE IF NOT EXISTS posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    title TEXT NOT NULL,
    content TEXT,
    status TEXT DEFAULT 'draft',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users (id)
  )
`;

// Database manager
export class DatabaseManager {
  private static store: relationalStore.RdbStore | null = null;

  static async initialize(context: Context): Promise<void> {
    try {
      this.store = await relationalStore.getRdbStore(context, DB_CONFIG);
      await this.createTables();
      console.log("Database initialized successfully");
    } catch (error) {
      console.error("Failed to initialize database:", error);
      throw error;
    }
  }

  private static async createTables(): Promise<void> {
    if (!this.store) {
      throw new Error("Database not initialized");
    }

    try {
      await this.store.executeSql(CREATE_USERS_TABLE);
      await this.store.executeSql(CREATE_POSTS_TABLE);
      console.log("Database tables created successfully");
    } catch (error) {
      console.error("Failed to create tables:", error);
      throw error;
    }
  }

  static getStore(): relationalStore.RdbStore {
    if (!this.store) {
      throw new Error("Database not initialized");
    }
    return this.store;
  }
}

// User data access object
export class UserDAO {
  static async createUser(
    user: Omit<User, "id" | "createdAt" | "updatedAt">
  ): Promise<number> {
    const store = DatabaseManager.getStore();

    try {
      const valueBucket: relationalStore.ValuesBucket = {
        name: user.name,
        email: user.email,
        age: user.age,
      };

      const rowId = await store.insert("users", valueBucket);
      console.log(`User created with ID: ${rowId}`);
      return rowId;
    } catch (error) {
      console.error("Failed to create user:", error);
      throw error;
    }
  }

  static async getUserById(id: number): Promise<User | null> {
    const store = DatabaseManager.getStore();

    try {
      const predicates = new relationalStore.RdbPredicates("users");
      predicates.equalTo("id", id);

      const resultSet = await store.query(predicates);

      if (resultSet.goToFirstRow()) {
        const user: User = {
          id: resultSet.getLong(resultSet.getColumnIndex("id")),
          name: resultSet.getString(resultSet.getColumnIndex("name")),
          email: resultSet.getString(resultSet.getColumnIndex("email")),
          age: resultSet.getLong(resultSet.getColumnIndex("age")),
          createdAt: resultSet.getString(
            resultSet.getColumnIndex("created_at")
          ),
          updatedAt: resultSet.getString(
            resultSet.getColumnIndex("updated_at")
          ),
        };

        resultSet.close();
        return user;
      }

      resultSet.close();
      return null;
    } catch (error) {
      console.error(`Failed to get user by ID ${id}:`, error);
      throw error;
    }
  }

  static async getAllUsers(): Promise<User[]> {
    const store = DatabaseManager.getStore();

    try {
      const predicates = new relationalStore.RdbPredicates("users");
      const resultSet = await store.query(predicates);

      const users: User[] = [];

      if (resultSet.goToFirstRow()) {
        do {
          const user: User = {
            id: resultSet.getLong(resultSet.getColumnIndex("id")),
            name: resultSet.getString(resultSet.getColumnIndex("name")),
            email: resultSet.getString(resultSet.getColumnIndex("email")),
            age: resultSet.getLong(resultSet.getColumnIndex("age")),
            createdAt: resultSet.getString(
              resultSet.getColumnIndex("created_at")
            ),
            updatedAt: resultSet.getString(
              resultSet.getColumnIndex("updated_at")
            ),
          };
          users.push(user);
        } while (resultSet.goToNextRow());
      }

      resultSet.close();
      return users;
    } catch (error) {
      console.error("Failed to get all users:", error);
      throw error;
    }
  }

  static async updateUser(
    id: number,
    updates: Partial<User>
  ): Promise<boolean> {
    const store = DatabaseManager.getStore();

    try {
      const valueBucket: relationalStore.ValuesBucket = {
        ...updates,
        updated_at: new Date().toISOString(),
      };

      const predicates = new relationalStore.RdbPredicates("users");
      predicates.equalTo("id", id);

      const rowsAffected = await store.update(valueBucket, predicates);
      return rowsAffected > 0;
    } catch (error) {
      console.error(`Failed to update user ${id}:`, error);
      throw error;
    }
  }

  static async deleteUser(id: number): Promise<boolean> {
    const store = DatabaseManager.getStore();

    try {
      const predicates = new relationalStore.RdbPredicates("users");
      predicates.equalTo("id", id);

      const rowsAffected = await store.delete(predicates);
      return rowsAffected > 0;
    } catch (error) {
      console.error(`Failed to delete user ${id}:`, error);
      throw error;
    }
  }

  static async searchUsers(query: string): Promise<User[]> {
    const store = DatabaseManager.getStore();

    try {
      const predicates = new relationalStore.RdbPredicates("users");
      predicates.like("name", `%${query}%`).or().like("email", `%${query}%`);

      const resultSet = await store.query(predicates);
      const users: User[] = [];

      if (resultSet.goToFirstRow()) {
        do {
          const user: User = {
            id: resultSet.getLong(resultSet.getColumnIndex("id")),
            name: resultSet.getString(resultSet.getColumnIndex("name")),
            email: resultSet.getString(resultSet.getColumnIndex("email")),
            age: resultSet.getLong(resultSet.getColumnIndex("age")),
            createdAt: resultSet.getString(
              resultSet.getColumnIndex("created_at")
            ),
            updatedAt: resultSet.getString(
              resultSet.getColumnIndex("updated_at")
            ),
          };
          users.push(user);
        } while (resultSet.goToNextRow());
      }

      resultSet.close();
      return users;
    } catch (error) {
      console.error(`Failed to search users with query "${query}":`, error);
      throw error;
    }
  }
}

// Data models
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  createdAt: string;
  updatedAt: string;
}

interface Post {
  id: number;
  userId: number;
  title: string;
  content: string;
  status: "draft" | "published" | "archived";
  createdAt: string;
  updatedAt: string;
}
```

## File Storage Operations

### File System Management

```typescript
import fs from "@ohos.file.fs";
import { Context } from "@ohos.ability.common";

export class FileStorageManager {
  private static filesDir: string = "";
  private static cacheDir: string = "";

  static initialize(context: Context): void {
    this.filesDir = context.filesDir;
    this.cacheDir = context.cacheDir;
  }

  // Write text file
  static async writeTextFile(
    fileName: string,
    content: string,
    isCache: boolean = false
  ): Promise<void> {
    const dirPath = isCache ? this.cacheDir : this.filesDir;
    const filePath = `${dirPath}/${fileName}`;

    try {
      const file = fs.openSync(
        filePath,
        fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE
      );
      await fs.write(file.fd, content);
      fs.closeSync(file);

      console.log(`Text file written: ${filePath}`);
    } catch (error) {
      console.error(`Failed to write text file ${fileName}:`, error);
      throw error;
    }
  }

  // Read text file
  static async readTextFile(
    fileName: string,
    isCache: boolean = false
  ): Promise<string> {
    const dirPath = isCache ? this.cacheDir : this.filesDir;
    const filePath = `${dirPath}/${fileName}`;

    try {
      if (!fs.accessSync(filePath)) {
        throw new Error(`File does not exist: ${filePath}`);
      }

      const file = fs.openSync(filePath, fs.OpenMode.READ_ONLY);
      const arrayBuffer = new ArrayBuffer(1024 * 1024); // 1MB buffer
      const readResult = await fs.read(file.fd, arrayBuffer);
      fs.closeSync(file);

      const content = String.fromCharCode(
        ...new Uint8Array(arrayBuffer, 0, readResult.bytesRead)
      );
      return content;
    } catch (error) {
      console.error(`Failed to read text file ${fileName}:`, error);
      throw error;
    }
  }

  // Write JSON file
  static async writeJsonFile<T>(
    fileName: string,
    data: T,
    isCache: boolean = false
  ): Promise<void> {
    try {
      const jsonContent = JSON.stringify(data, null, 2);
      await this.writeTextFile(fileName, jsonContent, isCache);
    } catch (error) {
      console.error(`Failed to write JSON file ${fileName}:`, error);
      throw error;
    }
  }

  // Read JSON file
  static async readJsonFile<T>(
    fileName: string,
    isCache: boolean = false
  ): Promise<T> {
    try {
      const content = await this.readTextFile(fileName, isCache);
      return JSON.parse(content) as T;
    } catch (error) {
      console.error(`Failed to read JSON file ${fileName}:`, error);
      throw error;
    }
  }

  // Copy file
  static async copyFile(
    sourceFileName: string,
    destFileName: string,
    sourceIsCache: boolean = false,
    destIsCache: boolean = false
  ): Promise<void> {
    const sourceDirPath = sourceIsCache ? this.cacheDir : this.filesDir;
    const destDirPath = destIsCache ? this.cacheDir : this.filesDir;
    const sourcePath = `${sourceDirPath}/${sourceFileName}`;
    const destPath = `${destDirPath}/${destFileName}`;

    try {
      await fs.copyFile(sourcePath, destPath);
      console.log(`File copied from ${sourcePath} to ${destPath}`);
    } catch (error) {
      console.error(
        `Failed to copy file from ${sourceFileName} to ${destFileName}:`,
        error
      );
      throw error;
    }
  }

  // Delete file
  static async deleteFile(
    fileName: string,
    isCache: boolean = false
  ): Promise<void> {
    const dirPath = isCache ? this.cacheDir : this.filesDir;
    const filePath = `${dirPath}/${fileName}`;

    try {
      if (fs.accessSync(filePath)) {
        fs.unlinkSync(filePath);
        console.log(`File deleted: ${filePath}`);
      }
    } catch (error) {
      console.error(`Failed to delete file ${fileName}:`, error);
      throw error;
    }
  }

  // List files in directory
  static async listFiles(isCache: boolean = false): Promise<string[]> {
    const dirPath = isCache ? this.cacheDir : this.filesDir;

    try {
      const files = fs.listFileSync(dirPath);
      return files;
    } catch (error) {
      console.error("Failed to list files:", error);
      throw error;
    }
  }

  // Get file info
  static async getFileInfo(
    fileName: string,
    isCache: boolean = false
  ): Promise<fs.Stat> {
    const dirPath = isCache ? this.cacheDir : this.filesDir;
    const filePath = `${dirPath}/${fileName}`;

    try {
      const stat = fs.statSync(filePath);
      return stat;
    } catch (error) {
      console.error(`Failed to get file info for ${fileName}:`, error);
      throw error;
    }
  }

  // Create directory
  static async createDirectory(
    dirName: string,
    isCache: boolean = false
  ): Promise<void> {
    const baseDirPath = isCache ? this.cacheDir : this.filesDir;
    const dirPath = `${baseDirPath}/${dirName}`;

    try {
      if (!fs.accessSync(dirPath)) {
        fs.mkdirSync(dirPath);
        console.log(`Directory created: ${dirPath}`);
      }
    } catch (error) {
      console.error(`Failed to create directory ${dirName}:`, error);
      throw error;
    }
  }
}

// Document manager for structured file storage
export class DocumentManager {
  private static readonly DOCUMENTS_DIR = "documents";
  private static readonly IMAGES_DIR = "images";
  private static readonly TEMP_DIR = "temp";

  static async initialize(): Promise<void> {
    await FileStorageManager.createDirectory(this.DOCUMENTS_DIR);
    await FileStorageManager.createDirectory(this.IMAGES_DIR);
    await FileStorageManager.createDirectory(this.TEMP_DIR, true); // Temp in cache
  }

  static async saveDocument(docId: string, content: any): Promise<void> {
    const fileName = `${this.DOCUMENTS_DIR}/${docId}.json`;
    await FileStorageManager.writeJsonFile(fileName, content);
  }

  static async loadDocument<T>(docId: string): Promise<T | null> {
    try {
      const fileName = `${this.DOCUMENTS_DIR}/${docId}.json`;
      return await FileStorageManager.readJsonFile<T>(fileName);
    } catch (error) {
      console.log(`Document ${docId} not found`);
      return null;
    }
  }

  static async deleteDocument(docId: string): Promise<void> {
    const fileName = `${this.DOCUMENTS_DIR}/${docId}.json`;
    await FileStorageManager.deleteFile(fileName);
  }

  static async listDocuments(): Promise<string[]> {
    const files = await FileStorageManager.listFiles();
    return files
      .filter(
        (file) => file.startsWith(this.DOCUMENTS_DIR) && file.endsWith(".json")
      )
      .map((file) =>
        file.replace(`${this.DOCUMENTS_DIR}/`, "").replace(".json", "")
      );
  }
}
```

## Best Practices

### 1. Choose the Right Storage Solution

- **Preferences**: For simple key-value data like user settings
- **RDB**: For complex relational data requiring queries
- **File Storage**: For large documents, images, or binary data
- **KV Store**: For distributed applications requiring data sync

### 2. Data Security

- Use appropriate security levels for sensitive data
- Implement proper encryption for critical information
- Validate data before storage operations
- Handle storage permissions properly

### 3. Performance Optimization

- Implement data caching strategies
- Use batch operations for multiple records
- Optimize database queries with proper indexing
- Clean up temporary files regularly

### 4. Error Handling

- Implement comprehensive error handling
- Provide fallback mechanisms for storage failures
- Log storage operations for debugging
- Handle storage quota limitations

## Conclusion

HarmonyOS provides comprehensive data storage solutions for different application needs. By understanding the strengths and use cases of each storage mechanism, you can design efficient and reliable data persistence strategies for your applications.

Key takeaways:

1. **Storage Selection**: Choose the appropriate storage mechanism based on data complexity and access patterns
2. **Data Organization**: Structure your data models and access patterns efficiently
3. **Error Handling**: Implement robust error handling and recovery mechanisms
4. **Performance**: Optimize storage operations for better user experience
5. **Security**: Apply appropriate security measures for sensitive data
