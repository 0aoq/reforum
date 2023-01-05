# reforum

Open-source forum app build on [PocketBase](https://pocketbase.io)

Required PocketBase schema can be found in `./pocketbase/pb_schema.json`

## Features

- Individual topics that can be created by users
    - User ability to create topics can be toggled (soon)
    - Topics can be deleted by the topic creator
    - Posts can be created within topics
        - Users can reply to posts
            - Replies can be deleted by the topic creator, post creator or reply sender
        - Posts can be deleted by the topic creator or post creator
- User profiles
    - User avatar
    - User bio
    - User rank in forum

## Repository Structure

- Connection management page: `./src/routes/+page.svelte`
- Client Server Connection: `./src/routes/[host]/`
    - Server auth: `./src/routes/[host]/auth`
    - Server topic view: `./src/routes/[host]/+page.svelte`
        - Server topic posts view: `./src/routes/[host]/t/[topicid]/+page.svelte`
            - Topic post view: `./src/routes/[host]/t/[topicid]/[postid]/+page.svelte`
    - User profile  view: `./src/routes/[host]/u/[username]/+page.svelte`
- Reusable components: `./src/routes/lib/components`
    - Site foorter: `./src/routes/lib/components/Footer.svelte`
    - Message (posts, replies, etc): `./src/routes/lib/components/Message.svelte`
        - _Note_: Also handles all functionality (message deletion, message permissions)