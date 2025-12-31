# Twenty CRM - Workspace Entity Patterns

**READ THIS FIRST** before creating any new objects.

This document explains how to add standard objects to Twenty by creating workspace entities in code, the same way Twenty defines People, Companies, and Opportunities.

## How Twenty's Object System Works

Twenty uses a **metadata-driven** approach:

1. **Workspace Entities** are TypeScript classes with decorators that define the schema
2. When you run `workspace:sync-metadata`, Twenty reads these definitions and:
   - Creates database tables
   - Generates GraphQL schema
   - Builds the UI automatically
3. Objects appear in Settings > Data Model and are usable immediately

**This is how we add our objects** - as new standard objects in code, not via UI or API.

---

## Step 1: Study Existing Standard Objects

Before creating anything, study these directories:

```
packages/twenty-server/src/modules/
├── person/                    # Person standard object
│   ├── standard-objects/
│   │   └── person.workspace-entity.ts
│   └── person.module.ts
├── company/                   # Company standard object  
│   ├── standard-objects/
│   │   └── company.workspace-entity.ts
│   └── company.module.ts
└── opportunity/               # Opportunity standard object
    ├── standard-objects/
    │   └── opportunity.workspace-entity.ts
    └── opportunity.module.ts
```

**Key file**: `*.workspace-entity.ts` - This is where object schema is defined.

---

## Step 2: Workspace Entity Structure

Here's the anatomy of a workspace entity (using Person as example):

```typescript
import { Relation } from 'src/engine/workspace-manager/workspace-sync-metadata/interfaces/relation.interface';
import { FieldMetadataType } from 'src/engine/metadata-modules/field-metadata/field-metadata.entity';
import { RelationMetadataType } from 'src/engine/metadata-modules/relation-metadata/relation-metadata.entity';
import {
  WorkspaceEntity,
  WorkspaceField,
  WorkspaceRelation,
  WorkspaceIsNullable,
  WorkspaceIsSystem,
} from 'src/engine/twenty-orm/decorators';
import { BaseWorkspaceEntity } from 'src/engine/twenty-orm/base.workspace-entity';

@WorkspaceEntity({
  standardId: 'PERSON_STANDARD_ID',  // Unique ID for this object
  nameSingular: 'person',
  namePlural: 'people', 
  labelSingular: 'Person',
  labelPlural: 'People',
  description: 'A person',
  icon: 'IconUser',
})
export class PersonWorkspaceEntity extends BaseWorkspaceEntity {
  
  @WorkspaceField({
    standardId: 'PERSON_NAME_FIELD_ID',
    type: FieldMetadataType.TEXT,
    label: 'Name',
    description: 'Person name',
    icon: 'IconUser',
  })
  name: string;

  @WorkspaceField({
    standardId: 'PERSON_EMAIL_FIELD_ID',
    type: FieldMetadataType.EMAIL,
    label: 'Email',
    description: 'Email address',
    icon: 'IconMail',
  })
  @WorkspaceIsNullable()
  email: string | null;

  @WorkspaceRelation({
    standardId: 'PERSON_COMPANY_RELATION_ID',
    type: RelationMetadataType.MANY_TO_ONE,
    label: 'Company',
    description: 'Company this person works for',
    icon: 'IconBuilding',
    inverseSideTarget: () => CompanyWorkspaceEntity,
    inverseSideFieldKey: 'people',
  })
  @WorkspaceIsNullable()
  company: Relation<CompanyWorkspaceEntity> | null;
}
```

### Key Decorators

| Decorator | Purpose |
|-----------|---------|
| `@WorkspaceEntity` | Defines the object (name, icon, description) |
| `@WorkspaceField` | Defines a field with type and metadata |
| `@WorkspaceRelation` | Defines a relationship to another entity |
| `@WorkspaceIsNullable` | Marks field as optional |
| `@WorkspaceIsSystem` | Marks as system field (hidden from UI) |

### Field Types

```typescript
import { FieldMetadataType } from 'src/engine/metadata-modules/field-metadata/field-metadata.entity';

FieldMetadataType.TEXT           // Short text
FieldMetadataType.RICH_TEXT      // Long formatted text
FieldMetadataType.NUMBER         // Integer
FieldMetadataType.CURRENCY       // Money with currency code
FieldMetadataType.DATE           // Date only
FieldMetadataType.DATE_TIME      // Date and time
FieldMetadataType.BOOLEAN        // True/false
FieldMetadataType.SELECT         // Single option
FieldMetadataType.MULTI_SELECT   // Multiple options
FieldMetadataType.RELATION       // Link to another object
FieldMetadataType.LINK           // URL
FieldMetadataType.EMAIL          // Email address
FieldMetadataType.PHONE          // Phone number
```

### Relation Types

```typescript
import { RelationMetadataType } from 'src/engine/metadata-modules/relation-metadata/relation-metadata.entity';

RelationMetadataType.ONE_TO_MANY   // This object has many of target
RelationMetadataType.MANY_TO_ONE   // Many of this belong to one target
RelationMetadataType.MANY_TO_MANY  // Many-to-many via junction
```

---

## Step 3: Creating a New Standard Object

### Example: Creating Firm Object

1. **Create directory structure**:
```
packages/twenty-server/src/modules/firm/
├── standard-objects/
│   └── firm.workspace-entity.ts
└── firm.module.ts
```

2. **Create workspace entity** (`firm.workspace-entity.ts`):
```typescript
import { Relation } from 'src/engine/workspace-manager/workspace-sync-metadata/interfaces/relation.interface';
import { FieldMetadataType } from 'src/engine/metadata-modules/field-metadata/field-metadata.entity';
import { RelationMetadataType } from 'src/engine/metadata-modules/relation-metadata/relation-metadata.entity';
import {
  WorkspaceEntity,
  WorkspaceField,
  WorkspaceRelation,
  WorkspaceIsNullable,
} from 'src/engine/twenty-orm/decorators';
import { BaseWorkspaceEntity } from 'src/engine/twenty-orm/base.workspace-entity';
import { FIRM_STANDARD_FIELD_IDS } from 'src/modules/firm/standard-objects/firm.standard-field-ids';

@WorkspaceEntity({
  standardId: FIRM_STANDARD_FIELD_IDS.FIRM,
  nameSingular: 'firm',
  namePlural: 'firms',
  labelSingular: 'Firm',
  labelPlural: 'Firms',
  description: 'A VC firm',
  icon: 'IconBuilding',
})
export class FirmWorkspaceEntity extends BaseWorkspaceEntity {

  @WorkspaceField({
    standardId: FIRM_STANDARD_FIELD_IDS.FIRM_EMAIL,
    type: FieldMetadataType.TEXT,
    label: 'Firm Email Domain',
    description: 'Email domain identifier (e.g., sequoiacap.com)',
    icon: 'IconMail',
  })
  firmEmail: string;

  @WorkspaceField({
    standardId: FIRM_STANDARD_FIELD_IDS.NAME,
    type: FieldMetadataType.TEXT,
    label: 'Name',
    description: 'Firm display name',
    icon: 'IconBuilding',
  })
  name: string;

  @WorkspaceField({
    standardId: FIRM_STANDARD_FIELD_IDS.WEBSITE,
    type: FieldMetadataType.LINK,
    label: 'Website',
    description: 'Firm website',
    icon: 'IconWorld',
  })
  @WorkspaceIsNullable()
  website: string | null;

  @WorkspaceField({
    standardId: FIRM_STANDARD_FIELD_IDS.STAGE_FOCUS,
    type: FieldMetadataType.MULTI_SELECT,
    label: 'Stage Focus',
    description: 'Investment stages',
    icon: 'IconTarget',
    options: [
      { value: 'PRE_SEED', label: 'Pre-seed', color: 'blue' },
      { value: 'SEED', label: 'Seed', color: 'green' },
      { value: 'SERIES_A', label: 'Series A', color: 'yellow' },
      { value: 'SERIES_B', label: 'Series B', color: 'orange' },
      { value: 'GROWTH', label: 'Growth', color: 'red' },
    ],
  })
  @WorkspaceIsNullable()
  stageFocus: string[] | null;

  // Relation: Firm has many People
  @WorkspaceRelation({
    standardId: FIRM_STANDARD_FIELD_IDS.PEOPLE,
    type: RelationMetadataType.ONE_TO_MANY,
    label: 'People',
    description: 'People at this firm',
    icon: 'IconUsers',
    inverseSideTarget: () => PersonWorkspaceEntity,
    inverseSideFieldKey: 'currentFirm',
  })
  people: Relation<PersonWorkspaceEntity[]>;
}
```

3. **Create standard field IDs** (`firm.standard-field-ids.ts`):
```typescript
// Generate UUIDs for each field - these must be stable
export const FIRM_STANDARD_FIELD_IDS = {
  FIRM: '20202020-0000-0000-0000-000000000001',
  FIRM_EMAIL: '20202020-0000-0000-0000-000000000002',
  NAME: '20202020-0000-0000-0000-000000000003',
  WEBSITE: '20202020-0000-0000-0000-000000000004',
  STAGE_FOCUS: '20202020-0000-0000-0000-000000000005',
  PEOPLE: '20202020-0000-0000-0000-000000000006',
};
```

4. **Create module** (`firm.module.ts`):
```typescript
import { Module } from '@nestjs/common';

@Module({})
export class FirmModule {}
```

5. **Register in parent module** (find where other standard objects are registered)

6. **Run sync**:
```bash
yarn command:dev workspace:sync-metadata -f
```

---

## Step 4: Extending Existing Objects

To add fields to Person (for current_firm, role, telegram_handle):

1. **Find the Person workspace entity**: `packages/twenty-server/src/modules/person/standard-objects/person.workspace-entity.ts`

2. **Add new fields**:
```typescript
// Add to PersonWorkspaceEntity class

@WorkspaceField({
  standardId: PERSON_STANDARD_FIELD_IDS.ROLE,
  type: FieldMetadataType.SELECT,
  label: 'Role',
  description: 'VC role',
  icon: 'IconBriefcase',
  options: [
    { value: 'PARTNER', label: 'Partner', color: 'blue' },
    { value: 'PRINCIPAL', label: 'Principal', color: 'green' },
    { value: 'ASSOCIATE', label: 'Associate', color: 'yellow' },
    { value: 'ANALYST', label: 'Analyst', color: 'gray' },
    { value: 'VENTURE_PARTNER', label: 'Venture Partner', color: 'purple' },
  ],
})
@WorkspaceIsNullable()
role: string | null;

@WorkspaceField({
  standardId: PERSON_STANDARD_FIELD_IDS.TELEGRAM_HANDLE,
  type: FieldMetadataType.TEXT,
  label: 'Telegram',
  description: 'Telegram handle',
  icon: 'IconBrandTelegram',
})
@WorkspaceIsNullable()
telegramHandle: string | null;

@WorkspaceRelation({
  standardId: PERSON_STANDARD_FIELD_IDS.CURRENT_FIRM,
  type: RelationMetadataType.MANY_TO_ONE,
  label: 'Current Firm',
  description: 'VC firm this person works for',
  icon: 'IconBuilding',
  inverseSideTarget: () => FirmWorkspaceEntity,
  inverseSideFieldKey: 'people',
})
@WorkspaceIsNullable()
currentFirm: Relation<FirmWorkspaceEntity> | null;
```

3. **Add IDs to person.standard-field-ids.ts**

4. **Run sync**

---

## Step 5: Select Fields with Options

For FirmEngagement status (21 options):

```typescript
@WorkspaceField({
  standardId: FIRM_ENGAGEMENT_FIELD_IDS.STATUS,
  type: FieldMetadataType.SELECT,
  label: 'Status',
  description: 'Engagement status',
  icon: 'IconProgress',
  options: [
    // Pipeline
    { value: 'PROSPECTING', label: 'Prospecting', color: 'gray', position: 0 },
    { value: 'WARM_INTRO_NEEDED', label: 'Warm Intro Needed', color: 'blue', position: 1 },
    { value: 'WARM_INTRO_SENT', label: 'Warm Intro Sent', color: 'blue', position: 2 },
    { value: 'COLD_OUTREACH_SENT', label: 'Cold Outreach Sent', color: 'blue', position: 3 },
    { value: 'FIRST_MEETING_SCHEDULED', label: '1st Meeting Scheduled', color: 'yellow', position: 4 },
    { value: 'FIRST_MEETING_COMPLETED', label: '1st Meeting Completed', color: 'yellow', position: 5 },
    { value: 'PARTNER_MEETING', label: 'Partner Meeting', color: 'orange', position: 6 },
    { value: 'DD', label: 'DD', color: 'orange', position: 7 },
    { value: 'INVESTMENT_COMMITTEE', label: 'Investment Committee', color: 'red', position: 8 },
    { value: 'TERM_SHEET', label: 'Term Sheet', color: 'red', position: 9 },
    { value: 'COMMITTED', label: 'Committed', color: 'green', position: 10 },
    // Special
    { value: 'WENT_COLD', label: 'Went Cold - Need Follow Up', color: 'purple', position: 11 },
    { value: 'WANT_LEAD', label: 'Want Lead', color: 'pink', position: 12 },
    { value: 'ON_ICE', label: 'On Ice', color: 'gray', position: 13 },
    // Rejections
    { value: 'REJECTED_GENERIC', label: 'Rejected - Generic', color: 'gray', position: 14 },
    { value: 'REJECTED_VALUATION', label: 'Rejected - Too High Val', color: 'gray', position: 15 },
    { value: 'REJECTED_GEO', label: 'Rejected - Not Fit Geo', color: 'gray', position: 16 },
    { value: 'REJECTED_THESIS', label: 'Rejected - Thesis', color: 'gray', position: 17 },
    { value: 'REJECTED_COMPETITOR', label: 'Rejected - Competitor', color: 'gray', position: 18 },
    { value: 'REJECTED_POST_DD', label: 'Rejected - Post DD/IC', color: 'gray', position: 19 },
    { value: 'REJECTED_TRACTION', label: 'Rejected - Post Traction', color: 'gray', position: 20 },
    { value: 'AVOID', label: 'Avoid', color: 'red', position: 21 },
  ],
  defaultValue: "'PROSPECTING'",
})
status: string;
```

---

## Step 6: After Creating Entities

1. **Sync metadata**:
```bash
yarn command:dev workspace:sync-metadata -f
```

2. **Verify in UI**: Go to Settings > Data Model - objects should appear

3. **Test CRUD**: Create a record via UI to verify fields work

4. **GraphQL**: Test queries at http://localhost:3000/graphql

---

## Workflows (Configured in UI)

After objects exist, configure workflows in Settings > Workflows:

### Trigger Types
- **Record is Created/Updated/Deleted**: Fires on record changes
- **On a Schedule**: Cron-based
- **Webhook**: External trigger
- **Manual**: User-initiated

### Action Types
- **Create/Update/Delete Record**
- **Send Email**
- **HTTP Request** (for transcript processing webhook)
- **Code** (custom TypeScript)

---

## Common Gotchas

1. **Standard IDs must be unique UUIDs** - Generate once, never change
2. **Run sync-metadata after every change** - Or changes won't apply
3. **Field names are camelCased** in GraphQL (firmEmail, not firm_email)
4. **Relations need inverse side** - Define both sides of relationship
5. **Options need position** - For consistent ordering in UI

---

## Testing

### GraphQL Playground
Access at: http://localhost:3000/graphql

### Check Objects Exist
```graphql
query {
  firms(first: 5) {
    edges { node { id name firmEmail } }
  }
}
```

### Verify Relations
```graphql
query {
  people(first: 5) {
    edges {
      node {
        id
        name
        currentFirm { name }
      }
    }
  }
}
```