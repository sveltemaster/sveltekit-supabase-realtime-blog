<script>
  import {readable, get} from 'svelte/store'
  import supabase from '$lib/supabase'
  import Post from '$lib/Post.svelte'

  async function addPost(e) {
    const formData = new FormData(e.target)
    const {data, error} = await supabase
      .from('posts')
      .insert({
        title: formData.get('title'),
        content: formData.get('content')
      })
  }


  const posts = readable(null, (set) => {
    supabase
      .from('posts')
      .select('*')
      .then(({error, data}) => set(data))

    const subscription = supabase
      .from('posts')
      .on('*', (payload) => {
        if (payload.eventType === 'INSERT') {
          set([...get(posts), payload.new])
        }
      })
      .subscribe()
      
    return () => supabase.removeSubscription(subscription)
  })
</script>

<main style="display: flex; flex-direction: column; align-items:center;">
  <h2>Make a Post</h2>
  <form on:submit|preventDefault={addPost}>
    <input placeholder=title input=text name=title>
    <br>
    <textarea placeholder=content name=content></textarea>
    <button>Add Post!</button>
  </form>

  <h2>See the Posts</h2>
  {#if $posts}
    {#each $posts as post}
      <Post {...post} />
    {/each}
  {:else}
    Loading..
  {/if}
</main>