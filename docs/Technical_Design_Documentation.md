# Technical Design Documentation

## Executive Summary

### App Definition and What is the Purpose

ERP Lite is a lightweight implementation of digitalized Enterprise Resource
Planning that contains well-known modules such as Procurement (for receiving
goods from suppliers). It is a software that runs on web platform or simply, a
web application.

The purpose is to demo how ERP module works, for a quick and easy to customize
manners, due to its simple technologies underneath.

Anyone can learn ERP in practical by just visiting the link, no installation
required, just your web browser.

### App Constraints

- Free and Open Source under MIT License, code lives on GitHub platform.
- in initial release or early stage, relational database will be implemented
  using web browser `localStorage`.
- Offline-first experience.
- Compatible on mobile phone.
- CAUTION! this software is not intended for real case day-to-day business as it
  does not implement a standard ERP system. For that case, I suggest using
  popular ERP softwares like SAP, Odoo or ERPNext.
- Procurement module will be released initially, other will follow for next
  releases, one module per release.

## The Technology Stack (RFC - Request for Comments)

**Core**

React, TypeScript

**UX**

Mantine

**Build**

Vite

**Testing**

- E2E: Cypress
- Integration & Unit: Testing Library, Vitest

**Routing**

React Router

**Data & State**

TanStack Query

## Design Decisions

### Business Flow

[Read here](./Business_Flows.md)

### Entity-Relationship Diagram

[Read here](./Entity_Relationship_Diagrams.md)

### Data layer

#### Custom ORM

API Expectation

```ts
// masterData.[Module Name].[Resource Name].[Action Methods]([Arguments for specific Action])
await masterData.Procurement.orders.getList("my supplier name", "descending");
```

Proposed

```ts
type RecordID = string;
type SortMethod = "descending" | "ascending";

/**
 * An interface describing response of a resource.
 */
interface RepositoryResponse<T> {
  data: T | null;
  success: boolean;
  errorCode: "NOT_FOUND" | "DUPLICATE" | "STORAGE_ERROR"; // Add more if needed
  /**
   * Message on success or error.
   */
  message: string;
}

/**
 * An interface describing operations on a specific Storage.
 */
interface IStorageDriver {
  connect(): Promise<void>;
  read(table: string): Promise<any>;
  write(table: string, data: any): Promise<void>;
}

/**
 * @remarks
 * T = Type of Data Model or Entity, e.g. Item, Receipt.
 */
abstract class BaseRepository<T extends { id: RecordID }> {
  protected db: IStorageDriver;

  /**
   * @param name - The name of the resource. The name may be similar to a Model/Entity name.
   */
  constructor(tableName: string, db: IStorageDriver) {
    this.tableName = tableName;
    this.db = db;
  }

  /**
   * Retrieve only one item based on its ID.
   *
   * @param id - ID of the item
   * @returns Data asked for further use
   */
  abstract getOne(
    id: RecordID,
  ): Promise<RepositoryResponse<T>>;
  /**
   * Retrieve more than 1 item.
   *
   * @param filter - Whether each item pass the check to be retrieved
   * @param sort - A method used for sorting the result
   * @returns Data asked for further use
   */
  abstract getList(
    filter?: Partial<T> | ((item: T) => boolean),
    sort?: SortMethod,
  ): Promise<RepositoryResponse<T[]>>;
  /**
   * Create a new item.
   *
   * @param data - Detail of new item to be added
   * @returns Newly generated ID of stored item
   */
  abstract add(data: Omit<T, "id">): Promise<RepositoryResponse<T>>;
  /**
   * Update an existing item.
   *
   * @param id - ID of existing item
   * @param data - Some item properties to be updated. Any excluded properties will remain the same as existing.
   * @returns New detail from updated item
   */
  abstract update(
    id: RecordID,
    data: Partial<T>,
  ): Promise<RepositoryResponse<T>>;
  /**
   * Delete an existing item.
   *
   * @param id - ID of existing item
   * @returns Is deletion success?
   */
  abstract delete(id: RecordID): Promise<RepositoryResponse<boolean>>;
}

/**
 * An interface describing a database with selected driver and high-level ORM.
 *
 * @remarks
 * This interface must be implemented once with an identifier named `masterData`
 */
interface Database {
  /**
   * Storage driver used.
   *
   * @remarks
   * This driver acts as low-level API to perform operations on a Storage (e.g. Web LocalStorage or REST API to a Backend)
   */
  _driver: IStorageDriver;

  // List of ERP Modules
  Procurement: {
    // List of Resources available on Procurement module
    orders: BaseRepository<PurchaseOrder>;
  };
  Inventory: {
    // List of Resources available on Inventory module
    items: BaseRepository<Material>;
  };
}
```

There is only 1 database connection and instance. For persistent relational DB,
the name can be `ERP_Lite`.

Table name must be written as `[Module Name]_[Entity]`. For example, Purchase
Order records/entries can be identified as `PurchaseOrder` and if it is owned by
Procurement module, then the table or collection name will be
`Procurement_PurchaseOrder`.
