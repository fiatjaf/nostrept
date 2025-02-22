<script>
  import {onMount} from 'svelte'

  import {emptyMetadata, sanitizeString} from '../lib/helpers'
  import {updateFollow, updateMetadata} from '../lib/actions'
  import {getMetadataStore} from '../lib/metadata'
  import {pubkey} from '../stores/state'
  import following from '../stores/following'
  import browsing from '../stores/browsing'
  import NoteCard from '../components/NoteCard.svelte'
  import ProfileName from '../components/ProfileName.svelte'

  export let params

  let meta, edit

  $: {
    meta = getMetadataStore(params.profile)
    edit = emptyMetadata()
    let unsub = meta.subscribe($meta => {
      edit = {...$meta}
      if (edit.name && edit.name.length > 0) {
        setTimeout(() => {
          unsub()
        }, 1)
      }
    })

    browsing.setPubkey(params.profile)
  }

  $: self = params.profile === pubkey
  $: followAction = $following.includes(params.profile) ? 'Unfollow' : 'Follow'

  async function setMetadata(e) {
    e.preventDefault()
    updateMetadata({
      picture: edit.picture && window.encodeURI(edit.picture.trim()),
      about: edit.about && sanitizeString(edit.about),
      name: edit.name && sanitizeString(edit.name),
      domain: edit.domain && sanitizeString(edit.domain)
    })
  }

  const handleFollow = ev => {
    ev.preventDefault()
    updateFollow(followAction.toLowerCase(), params.profile)
  }
</script>

<div class="px-4">
  <header class="block my-5">
    <h1 class="title">Profile</h1>

    <p>
      <code>{params.profile}</code>
      {#if self}
        <span class="icon">
          <i class="icon ion-md-person" />
        </span>
      {/if}
    </p>
    <p class="subtitle">
      {#if $meta.picture}
        <img alt="~" class="is-64x64 image" src={$meta.picture} />
      {/if}
      <ProfileName meta={$meta} />
    </p>
    <p>{$meta.about || ''}</p>
  </header>
  {#if self}
    <div class="block">
      <form>
        <div class="field">
          <div class="control">
            <label>
              Avatar
              <input
                class="input"
                type="text"
                placeholder="https://picture.example.com/me.png"
                bind:value={edit.picture}
              />
            </label>
          </div>
        </div>
        <div class="field">
          <div class="control">
            <label>
              Nickname
              <input
                class="input"
                type="text"
                placeholder="mynickname"
                bind:value={edit.name}
              />
            </label>
          </div>
        </div>
        <div class="field">
          <div class="control">
            <label>
              Domain
              <input
                class="input"
                type="text"
                placeholder="myname.com"
                bind:value={edit.domain}
              />
            </label>
          </div>
        </div>
        <div class="field">
          <div class="control">
            <label>
              Bio
              <textarea
                class="textarea"
                placeholder="I am x and I like y."
                bind:value={edit.about}
              />
            </label>
          </div>
        </div>
      </form>
    </div>
    <div class="block right">
      <button class="button is-primary" on:click={setMetadata}>Save</button>
    </div>
  {:else}
    <div class="block">
      <button class="button primary" on:click={handleFollow}
        >{followAction}</button
      >
    </div>
  {/if}
  <div class="block my-5">
    <h2 class="subtitle">Notes</h2>
    {#each $browsing.notes as note}
      <NoteCard {note} />
    {/each}
  </div>
</div>

<style>
  .right {
    display: flex;
    flex-flow: row-reverse;
  }
  code {
    padding: 0;
  }
</style>
