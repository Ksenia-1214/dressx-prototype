# Reference photo

Spec for the **Reference** block inside **Product Measurements**. It matches the current prototype.

A reference is an optional on-model photo. It tells generation how the garment should fit, how long it is, and where it should sit. It is not a catalog view, and it does not replace the Studio model. The Studio model is still the person in the final image.

---

## 1. Reference vs product photos

Product photos and the reference do different jobs. Do not merge them into one list.

| | Product photos | Reference |
|---|---|---|
| Purpose | What the garment looks like. The model wears exactly what these photos show. | How the garment sits on a body: fit, length, drape, placement. |
| Required | At least one photo. If there is more than one, one of them must be **Main**. | Optional. Saving never depends on it. |
| Roles | **Main**, **Back view**, **Secondary**, or **No role**. | No role. It is its own field. |
| Count | Up to 4 files on the product. | Exactly one. A new upload replaces the current one. |
| Shown in the final image as the wearer | No. These are garment sources. | No. The Studio model stays the wearer. |

**Good reference:** the item on a person or a mannequin, with enough of the garment visible to read length and how it sits (hem, waist, tuck, shoulder, sleeve).

**Not a reference:** flat lay, hanger packshot, folded garment, or a clean product view. Those belong in product photos. A mannequin shot can be either one — the difference is intent. If it is the catalog view of the item, it is a product photo. If it only shows how the item should sit, it is a reference.

---

## 2. Where it sits

Inside the Product Measurements modal, left column, under the product photo grid.

Two presentations:

- **Collapsed** (default when no reference is saved). One row: title, Optional pill, info button, short description, **Add**.
- **Expanded.** Title, description, guideline, and one of the three bodies below.

Opening a product that already has a reference opens the block expanded. **Hide** is available only while the block is empty. Once a reference is set, the block stays open until the user removes it.

The info button (**i**) toggles a short guideline under the row. It does not open a modal. **aria-label:** `Reference guidelines`.

---

## 3. States

### Empty

User can add a reference in two ways:

1. **Upload reference** — one file from the computer. Helper line: `From your computer`.
2. **Or choose from product photos without a role** — only if at least one product photo is set to **No role**. Opens a thumbnail grid of those photos. The toggle label becomes `Hide product photos` while the grid is open.

Choosing a product photo **copies** it into the reference. The original stays in the product grid, still **No role**. The same image can exist in both places. That is intentional: the product photo can later get a role, and the reference stays a separate input.

### Filled

- Thumbnail, 88×88, image contained (not cropped).
- Title: `Reference added`.
- Note: `Used for fit and placement on the model.`
- **Replace** uploads a new file and overwrites the current reference.
- **Remove** clears it. The block stays expanded so the user can add another one immediately.
- A remove control also sits on the thumbnail (`Remove reference`).

**Hide** is not shown in this state.

### Locked — no free slot

Shown only when there is no reference yet, every product photo has a role, and those roles already use all 4 slots (four assigned roles).

- Title: `No free slot`
- Body: `Set one photo to No role, or remove it from the product card.`
- Upload and the photo picker are hidden.
- The card background goes neutral (`#f6f6f8`), same family as the collapsed row.

To unlock: set one photo to **No role**, or remove a product photo. Then the empty state returns.

### Collapsed

Compact row on a neutral background, no visible border.

`Reference` · `Optional` · info · `On-model fit & placement` · **Add**

**Add** expands the block. It does not open the file picker by itself.

---

## 4. Slot budget

Roles and the reference **share 4 slots**. Photos left on **No role** do not use a slot.

```
slots used = (photos that have a role) + (1 if a reference is set, else 0)
```

The counter in the photo header shows this number: `2 / 4`, not the raw file count.

| Setup | Slots used | Roles still allowed |
|---|---|---|
| No reference | 0–4 | 4 |
| Reference set | roles + 1 | 3 |
| 4 roles, no reference | 4 | 0 — reference is locked |
| Reference + 3 roles | 4 | 0 new roles |

Rules that follow from this:

- A product can still hold **4 photo files** even when a reference is set. The 4th file must stay on **No role**. The reference does not reduce the file cap.
- Assigning a role past the budget does nothing and shows a toast.
  - No reference: `Only 4 photos can have roles`
  - Reference set: `Reference uses 1 of 4 slots — only 3 photos can have roles`
- Header hint, no reference: `Main is required. Back and Secondary are optional.` Hidden when the product has only one photo.
- Header hint, reference set: `Reference is set — assign roles to at most 3 photos (Main required). Extra photos stay as No role.`

Role limits, unchanged by the reference:

- **Main** — one photo. Required before save when the product has more than one photo. A single photo is saved as **Main** automatically.
- **Back view** — one photo. Optional. Setting it clears **Back view** on any other photo.
- **Secondary** — up to two photos. Optional. A third attempt shows `Only 2 Secondary photos allowed`.
- **No role** — stored on the product, not sent as a generation view, does not use a slot.

If a reference is added while too many roles are assigned, drop roles until the budget fits. Drop from the end of the list, in this order: **Secondary**, then **Back view**, then **Main**.

---

## 5. Actions

| Action | Result | Toast |
|---|---|---|
| Expand | Opens the empty or locked body. | — |
| Hide | Collapses the block. Only when no reference is set. | — |
| Upload | Sets the reference. One image. | `Reference added` |
| Replace | Overwrites the current reference. Allowed even when all 4 slots are full, because this reference already holds its slot. | `Reference replaced` |
| Pick a No-role photo | Copies that image into the reference. Source photo stays in the grid. | `Photo set as reference` |
| Remove | Clears the reference. Block stays expanded. | `Reference removed` |
| Upload or pick while locked | No change. | `Set one photo to No role, or remove it from the product card` |
| Pick while every photo already has a role | No change. | `Set one photo to No role first` |

Accepted files: **JPEG, PNG, WebP**. One file for the reference. The product-photo picker stays multi-file.

At least one product photo must remain. Removing the last one shows `Keep at least one product photo`. The reference can be removed on its own; that does not affect the photo count.

---

## 6. Save

Reference does not gate **Save Product**.

Save is blocked only when:

- the product has more than one photo and none of them is **Main**, or
- a required measurement for the category is empty.

On save, persist the reference beside the photos, not inside the role list:

```json
{
  "photos": [
    { "img": "<url>", "role": "Main" },
    { "img": "<url>", "role": "Back view" },
    { "img": "<url>", "role": "Secondary" },
    { "img": "<url>", "role": null }
  ],
  "reference": { "img": "<url>" }
}
```

`reference` is `null` when empty. `role: null` is **No role**.

The product photo count shown on the card is `photos.length`. It does not include the reference.

Generation input, when a pipeline is wired up:

- Send role-tagged photos as the garment sources (**Main** required, **Back view** and **Secondary** optional).
- Send `reference` as a separate fit-and-placement input.
- Do not send **No role** photos as views.
- Do not use the reference as the person in the frame. The Studio model stays the wearer.

---

## 7. Copy

Use these strings as written.

| Element | Copy |
|---|---|
| Title | Reference |
| Badge | Optional |
| Collapsed description | On-model fit & placement |
| Expanded description | On-model photo for fit and placement — not a product view. |
| Guideline title | What makes a good reference |
| Guideline body | A photo of the item on a model or mannequin — to show fit, length, and how it should sit. The Studio model is still used in the final image. |
| Primary action, collapsed | Add |
| Upload button | Upload reference |
| Upload helper | From your computer |
| Divider | or |
| Picker, closed | Or choose from product photos without a role |
| Picker, open | Hide product photos |
| Collapse | Hide |
| Filled title | Reference added |
| Filled note | Used for fit and placement on the model. |
| Replace | Replace |
| Remove | Remove |
| Locked title | No free slot |
| Locked body | Set one photo to No role, or remove it from the product card. |

---

## 8. Acceptance checks

1. A product with only **Main** saves with no reference.
2. Adding a reference changes the counter from `1 / 4` to `2 / 4` and the hint to the “at most 3 photos” line.
3. With a reference set, a fourth role is rejected and the existing three roles stay as they are.
4. Four assigned roles and no reference show **No free slot**. Clearing one role unlocks upload and, if that photo is **No role**, the picker.
5. Picking a **No role** photo fills the reference and leaves that photo in the grid.
6. **Replace** swaps the image and does not ask for a free slot.
7. **Remove** clears the image, shows `Reference removed`, and leaves the block expanded.
8. **Hide** is absent while a reference is set, and returns after remove.
9. A single product photo still saves as **Main** when a reference is attached.
10. JPEG, PNG, and WebP are accepted. Anything else is ignored.
