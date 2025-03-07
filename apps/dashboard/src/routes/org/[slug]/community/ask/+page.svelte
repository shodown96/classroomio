<script lang="ts">
  import { goto } from '$app/navigation';
  import ArrowLeftIcon from 'carbon-icons-svelte/lib/ArrowLeft.svelte';
  import { currentOrgPath, currentOrg } from '$lib/utils/store/org';
  import PrimaryButton from '$lib/components/PrimaryButton/index.svelte';
  import { askCommunityValidation } from '$lib/utils/functions/validator';
  import { supabase } from '$lib/utils/functions/supabase';
  import { snackbar } from '$lib/components/Snackbar/store';
  import generateSlug from '$lib/utils/functions/generateSlug';
  import TextEditor from '$lib/components/TextEditor/index.svelte';
  import TextField from '$lib/components/Form/TextField.svelte';

  let errors: {
    title?: string;
  } = {};
  let fields = {
    title: '',
    body: ''
  };

  async function handleSave() {
    errors = askCommunityValidation(fields);
    console.log('handleSave errors', errors);

    if (Object.keys(errors).length) {
      return;
    }

    const { data: question, error } = await supabase
      .from('community_question')
      .insert({
        ...fields,
        organization_id: $currentOrg.id,
        author_id: $currentOrg.memberId,
        votes: 0,
        slug: generateSlug(fields.title)
      })
      .select();

    if (error) {
      console.error('Error: asking question', error);
      snackbar.error('Error - Please try again later');
    } else {
      console.log('Success: asking question', question);
      snackbar.success('Success');
      goto(`${$currentOrgPath}/community/${question[0].slug}`);
    }
  }
</script>

<svelte:head>
  <title>Ask the Community - ClassroomIO</title>
</svelte:head>

<section class="w-full md:max-w-3xl md:mx-auto">
  <div class="p-5">
    <a
      class="text-gray-500 dark:text-white text-md flex items-center"
      href={`${$currentOrgPath}/community`}
    >
      <ArrowLeftIcon size={24} class="carbon-icon dark:text-white" /> Go Back
    </a>
    <div class="flex items-center gap-12 justify-between">
      <h1 class="dark:text-white text-2xl md:text-3xl font-bold">Ask the community</h1>
      <PrimaryButton label="Publish" onClick={handleSave} />
    </div>
  </div>

  <div class="mb-3 p-2">
    <TextField 
      bind:value={fields.title}
      placeholder="Title"
      errorMessage={errors.title}
    />
  </div>
  <div class="px-2">
    <TextEditor
      bind:value={fields.body}
      placeholder="Ask the community any question you might have"
      onChange={(html) => (fields.body = html)}
    />
  </div>
</section>
