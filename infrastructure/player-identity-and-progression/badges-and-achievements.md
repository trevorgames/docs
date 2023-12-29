# Badges & Achievements

## What are Badges & Achievements?

Badges & Achievements are in-game collectibles earned by playing, collecting, and competing in Trevor. They showcase your accomplishments and progress.

## Badge Technical Implementation

### **Step 1: Design Your Badge**

Before selecting a use case, review the Trevor Badge Design Guide for preparing image assets.

### **Step 2: Identify Your Use Case**

**(1) Host Badges on TrevorBadges contract, Trevor app handles claiming/display**

To quickly set up a badge without your Badge infrastructure:

* Provide a way to check eligibility (static list, API call, contract method, or possession of ERC721/ERC1155/ERC20 token).
* Send designed image assets following the Trevor Badge Guide.
* Provide metadata (REQUIRED section described below).

**(2) Host Badges tracking + claiming yourself, display on Trevor app**

* Best for decentralization.
* Send designed image assets following the Trevor Badge Guide.
* Provide metadata in both REQUIRED + OPTIONAL sections.

**(3) Host Badge tracking yourself, allow claiming/display of Badge from Trevor app**

* Decentralized hosting with the option to let users claim from the Trevor app.
* Provide a way to check eligibility (static list, API call, contract method, or possession of ERC721/ERC1155/ERC20 token).
* Send designed image assets following the Trevor Badge Guide.
* Provide metadata in both REQUIRED + OPTIONAL sections.
* Specify input parameters to call your badge contract for claiming.

### **Step 3: Provide Us Metadata About Your Badge**

The TypeScript schema below represents Badge configurations in our backend (subset):

```typescript
interface BadgeConfig {
  // REQUIRED: Basic info for the badge.
  name: string;
  description: string;
  image: string;
  rarity?: 'common' | 'rare' | 'legendary';
  // OPTIONAL: Additional metadata fields as needed
}
```

This schema serves as a basis for conveying Badge details to integrate with the Treasure ecosystem.
