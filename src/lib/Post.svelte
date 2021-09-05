<script>
  import supabase from '$lib/supabase'
  import {readable, get} from 'svelte/store'

  export let id
  export let title
  export let content

  async function addComment({target}) {
    const formData = new FormData(target)

    const {data, error} = await supabase
      .from('comments')
      .insert({
        content: formData.get('comment'),
        post: id
      })
  }

  const comments = readable(null, (set) => {
    supabase
      .from('comments')
      .select('*')
      .filter('post', 'eq', id)
      .then(({error, data}) => set(data))

    const subscription = supabase
      .from('comments:post=eq.' + id)
      .on('*', (payload) => {
        if (payload.eventType === 'INSERT') {
          set([...get(comments), payload.new])
        }
      })
      .subscribe()

    return () => supabase.removeSubscription(subscription)
  })
</script>

<article style="padding: 2rem;">
  <h3>{title}</h3>
  <p>{content}</p>
  <h4>comments</h4>
  {#if $comments}
    {#each $comments as {content}}
      <p>{content}</p>
    {:else}
      No comments yet!
    {/each}
  {:else}
    Loading..
  {/if}


  <form on:submit|preventDefault={addComment}>
    <input placeholder=comment name=comment>
    <button>Add Comment</button>
  </form>
</article>