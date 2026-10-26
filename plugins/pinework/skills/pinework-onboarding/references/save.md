# Save on a yes

Save nothing until the user says yes. If they change the proposal, save the changed version. A no saves nothing. Say so and move on.

## 1. Upload the source

Upload each source file once: the bundle from `pinework import knowledge`, and `paste.md` if there was a paste.

```
pinework file upload <bundle path> --description "Scrubbed knowledge bundle from <user>'s Mac, onboarding"
```

Keep each `file_…` id from the output.

## 2. Write the wiki pages

Write each page to a local file. Every fact line ends with the file it came from and the path inside it:

```
- shopfront is a Next.js 15 storefront for Rivera Ceramics. [pinework://file/file_…, ~/code/shopfront/CLAUDE.md]
```

A paste fact cites the paste file: `[pinework://file/file_…, paste]`.

```
pinework wiki create --title "About Sam" --body-file about-sam.md
pinework wiki create --title "shopfront" --body-file shopfront.md
```

Keep each page's `wik_…` id from the output.

## 3. Create the agents

Create each agent the user kept with the `create-agent` skill. In each agent's instructions, add a short section that links the pages it needs:

```
## Know
- [About Sam](pinework://wiki/wik_…)
- [shopfront](pinework://wiki/wik_…)
```

Link only the pages that agent's job needs. Never copy the facts into the instructions.

## 4. Tell the user

One line per agent: its name and @handle. One line for the pages: how many you saved. Then go to config.md, or to the hand-off.
