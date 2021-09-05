<script>
  import supabase from '$lib/supabase'
  import {readable, get} from 'svelte/store'
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

    // add subscription to supabase logic
    const subscription = supabase
      .from('posts')
      .on('INSERT', (payload) => {
        // payload.evenType
        // payload.new
        // payload.old
        set([...get(posts), payload.new]) 
      })
      .subscribe()


    return () => supabase.removeSubscription(subscription)
  })

</script>

<main style="display: flex; flex-direction: column; align-items:center;">
  <h1>Welcome to our blog!</h1>
  <h2>Make a Post</h2>
  <form on:submit|preventDefault={addPost}>
    <input placeholder=title name=title> <br>
    <textarea name=content></textarea> <br>
    <button>Make Post</button>
  </form>

  <h2>See your Posts</h2>
  {#if $posts}
    {#each $posts as {title, content, id}}
      <Post {title} {content} {id} />
    {/each}
  {:else}
    Loading..
  {/if}
</main>