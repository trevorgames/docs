# Quest Technical Implementation

\
For the seamless setup of a Quest on the Trevor Platform, we require the following details.&#x20;

**Quest Details:**

1. **Quest Name:**
2. **Associated Game:**
3. **Quest Description:** A concise, non-technical definition (preferably within a 75-character limit) of the actions a player must undertake to achieve this quest.
4. **Technical Quest Criteria:** Share contract info (contract address + function call parameters) to verify if a user with a specific `walletAddress`  has completed the quest.

```typescript
//  Response Format
interface Response {
  walletAddress: string;
  completed: boolean;
  numerator: number;
  denominator: number;
}
// Example Response JSON
{
  "walletAddress": "0x1234....",
  "completed": false,
  "numerator": 4,
  "denominator": 10
}
```

Here, `numerator` and `denominator` are used by our frontend to display partial progress.

5. **Required Items (Optional)**: Specify if the quest requires users to own a specific token/gamepiece.
6. **CTA Link:** URL directing players to navigate and work on the quest.
7. **Badge (Optional):** Attach a badge to accompany this specific quest.
8. **Quest Mechanics:**&#x20;
   1. **Friction Score:** On a scale of 1-10 (10 being the most friction), rate the user experience in completing this quest.
   2. **Difficulty Score:** On a scale of 1-10 (10 being the most difficult), assess the skill required for completing this quest.
   3. **Time Required:** Estimate the average time commitment (in hours) a player needs to complete this quest.

Your contributions are vital in enhancing the Trevor gaming experience. Thank you for your collaboration!
