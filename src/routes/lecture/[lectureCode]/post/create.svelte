<script context="module" lang="ts">
  import { checkPermission } from "$lib/api/auth";

  export async function load({ session, params }) {
    if (!session.auth) {
      return {
        status: 301,
        redirect: "/auth/login",
      };
    }

    const check = await checkPermission(session.auth, {
      userTypes: ["P"],
      lectureCode: params.lectureCode,
    });

    if (!check) {
      return {
        status: 301,
        redirect: `/lecture/${params.lectureCode}`,
      };
    }

    // console.log(params.lectureCode);
    return {
      props: {
        lectureCode: params.lectureCode,
      },
    };
  }
</script>

<script lang="ts">
  import Page from "$lib/shared/Page.svelte";
  import Button from "$lib/shared/Button.svelte";

  import { marked } from "marked";
  import sanitizeHtml from "sanitize-html";
  import type { PostForm } from "$lib/types";
  import { createPost } from "$lib/api/lectures";
  import { session } from "$app/stores";
  import { goto } from "$app/navigation";

  export let lectureCode: string;
  let inputField: PostForm = {
    title: "",
    body: "",
  };

  $: markdown = sanitizeHtml(marked(inputField.body));
  let error: string;

  let example = [
    "# H1 heading",
    "## H2 heading",
    "### H3 heading",
    "--------",
    "**bold text**",
    "*italicized text*",
    "--------",
    "1. First item\n2. Second item\n3. Third item",
    "- First item\n- Second item\n- Third item",
    "[Svelte](https://svelte.dev/)",
  ].join("\n\n");

  function seeExample() {
    inputField.body = example;
  }

  async function submit() {
    const res = await createPost($session["auth"], lectureCode, inputField);

    if (!res["error"]) goto(`/lecture/${lectureCode}`);
    error = res["error"];
  }
</script>

<div class="post-create">
  <Page title="Create Post" description="새로운 게시물을 작성할 수 있습니다.">
    {#if error}
      <p class="error">{error}</p>
    {/if}

    <div class="form-field">
      <input
        type="text"
        bind:value={inputField.title}
        placeholder="Post Title (ex: Hello Students)"
      />
    </div>

    <div class="markdown-editor">
      <div class="left-panel">
        <textarea bind:value={inputField.body} class="source" />
      </div>

      <div class="right-panel">
        <h2>Output</h2>
        <div class="output">{@html markdown}</div>
      </div>
    </div>

    <Button on:click={seeExample}>See Example</Button>
    <Button on:click={submit}>Create New Post</Button>
  </Page>
</div>

<style>
  .markdown-editor {
    width: 100%;
    height: 60vh;
    display: flex;
    align-items: flex-start;
    justify-content: space-evenly;
  }
  .left-panel,
  .right-panel {
    width: 50%;
    height: 100%;
    border: solid 1px black;
    background: #ffffff;
    padding: 10px 20px;
  }
  .right-panel {
    overflow: auto;
  }
  h2 {
    text-align: center;
  }
  textarea {
    resize: none;
  }
  .source {
    border: none;
    width: 100%;
    height: 100%;
  }
  .source:focus {
    outline: none;
  }
  .output {
    width: 100%;
    height: 100%;
  }
</style>
