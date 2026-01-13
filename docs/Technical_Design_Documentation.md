# Technical Design Documentation

## Executive Summary

### App Definition and What is the Purpose

ERP Lite is a lightweight implementation of digitalized Enterprise Resource Planning that contains well-known modules such as Procurement (for receiving goods from suppliers). It is a software that runs on web platform or simply, a web application.

The purpose is to demo how ERP module works, for a quick and easy to customize manners, due to its simple technologies underneath.

Anyone can learn ERP in practical by just visiting the link, no installation required, just your web browser.

### App Constraints

- Free and Open Source under MIT License, code lives on GitHub platform.
- in initial release or early stage, relational database will be implemented using web browser `localStorage`.
- Offline-first experience.
- Compatible on mobile phone.
- CAUTION! this software is not intended for real case day-to-day business as it does not implement a standard ERP system. For that case, I suggest using popular ERP softwares like SAP, Odoo or ERPNext.
- Procurement module will be released initially, other will follow for next releases, one module per release.

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

- [Procurement](./Business_Flows.md#Procurement)

### Entity-Relationship Diagram

Entity-Relationship Diagram or ERD or ER Model describes composed entities and specifies relationships between them, in a specific domain of knowledge.

Entity is

- [Procurement](./Entity_Relationship_Diagrams.md#Procurement)

### Data layer

#### Custom ORM

API Expectation

```ts
// masterData.[Module Name].[Resource Name].[Action Methods]([Arguments for specific Action])
await masterData.Procurement.orders.getList("my supplier name", "descending");
```

Proposed

```ts
type ResourceName = string;
type RecordID = string;
type SortMethod = "descending" | "ascending";

/**
 * An interface describing response of a resource.
 */
interface ResourceResponse<T> {
  data: T | null;
  success: boolean;
  /**
   * HTTP error status code.
   */
  errorCode: string | null;
  /**
   * Message on success or error.
   */
  message: string;
}

/**
 * @remarks
 * T = Type of Data Model or Entity, e.g. Item, Receipt.
 */
abstract class Resource {
  /**
   * @param name - The name of the resource. The name may be similar to a Model/Entity name.
   */
  constructor(name: ResourceName) {
    this.name = name;
  }

  /**
   * Retrieve only one item based on its ID.
   *
   * @param id - ID of the item
   * @param filter - A query line to make search be more specific
   * @returns Data asked for further use
   */
  abstract getOne<T>(
    id: RecordID,
    filter?: string
  ): Promise<ResourceResponse<T>>;
  /**
   * Retrieve more than 1 item.
   *
   * @param filter - A query line to shrink the result
   * @param sort - A method used for sorting the result
   * @returns Data asked for further use
   */
  abstract getList<T>(
    filter?: string,
    sort?: SortMethod
  ): Promise<ResourceResponse<T[]>>;
  /**
   * Create a new item.
   *
   * @param data - Detail of new item to be added
   * @returns Newly generated ID of stored item
   */
  abstract add<T>(data: T): Promise<ResourceResponse<RecordID>>;
  /**
   * Update an existing item.
   *
   * @param id - ID of existing item
   * @param data - Some item properties to be updated. Any excluded properties will remain the same as existing.
   * @returns New detail from updated item
   */
  abstract update<T>(id: RecordID, data: T): Promise<ResourceResponse<T>>;
  /**
   * Delete an existing item.
   *
   * @param id - ID of existing item
   * @returns Deleted item ID
   */
  abstract delete<T>(id: RecordID): Promise<ResourceResponse<RecordID>>;
}

/**
 * An interface describing a database detail and ORM.
 *
 * @remarks
 * This interface must be implemented once with an identifier named `masterData`
 */
interface Database {
  /**
   * The storage instance.
   *
   * @remarks
   * It acts as low level API that will be used by Resource instance to communicate. If `type` is "Browser" then this will hold `localStorage`
   */
  _storage: Storage | null;
  type: "Browser" | "Server SQL";

  /**
   * Connect to DB and set `storage`
   *
   * @returns - All of these
   */
  setup(): Promise<Database>;

  // List of ERP Modules
  Procurement: {
    // List of Resources available on Procurement module
    orders: Resource;
  };
  Inventory: {
    // List of Resources available on Inventory module
    items: Resource;
  };
}
```

There is only 1 database connection and instance. For persistent relational DB, the name can be `ERP_Lite`.

Table name must be written as `[Module Name]_[Entity]`. For example, Purchase Order records/entries want to be referred as `PurchaseOrder` and if it is owned by Procurement module, then the table or collection name must be `Procurement_PurchaseOrder`.
