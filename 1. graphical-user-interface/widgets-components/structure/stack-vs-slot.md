### When to Use Stack vs Slot

**Stacks** are rigid, predefined structures designed to impose a specific layout for example, arranging items in a row (`XStack`), a column (`YStack`), or depth perspective (`ZStack`). They always enforce their layout rules and are ideal when your content must follow a strict arrangement.

**Slots** are highly flexible structures. Use a Slot when you don’t know in advance how the content should be arranged, or when you want to programmatically control the layout with custom state or styles. Slots won’t constrain or override your layout decisions.

In short: Stacks represent certainty; Slots represent flexibility