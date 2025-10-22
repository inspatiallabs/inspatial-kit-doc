# Presentation

## Modal

```typescript
import { Modal } from "@inspatial/kit/presentation";
import { YStack } from "@inspatial/kit/structure";
import { Text } from "@inspatial/kit/typography";
import { Button } from "@inspatial/kit/ornament";

<Modal
  id="counter-modal"
  closeOnEsc
  closeOnScrim
  className="flex justify-center items-center h-screen w-screen m-auto"
>
  <YStack className="p-6 gap-3 w-[500px] h-[500px] bg-(--brand) rounded-3xl shadow-effect">
    <Text className="text-xl font-semibold">Title</Text>
    <Text>
      Use the buttons to adjust the counter and explore trigger props. This
      modal is controlled via on:presentation.
    </Text>
    <Button
      format="outline"
      on:presentation={{ id: "counter-modal", action: "close" }}
    >
      Close
    </Button>
  </YStack>
</Modal>;
```

## Presentation Portals

Uses the Presentation Portals `Inlet/Outlet` to render components outside their original DOM hierarchy.

```typescript
import {
  PresentationInlet,
  PresentationOutlet,
  Modal,
} from "@inspatial/kit/presentation";

<Modal>
  {/* The Presentation Outlet displays content provided by the Inlet */}
  <PresentationOutlet fallback={() => <Text>Hello World</Text>} />
</Modal>;

{
  /* The Presentation Inlet sends its children to be rendered by the Outlet */
}
<PresentationInlet>
  <Text>Render on the modal</Text>
</PresentationInlet>;
```
