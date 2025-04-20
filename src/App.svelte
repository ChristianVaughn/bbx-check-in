<!-- App.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  
  // Define interfaces
  interface Participant {
    id: string;
    name: string;
    checkedIn: boolean;
  }

  // Initial state
  let participants: Participant[] = [];
  let searchText: string = '';
  let showCheckedInList: boolean = false;
  let importText: string = '';
  let showImportModal: boolean = false;
  let showTextInput: boolean = false;
  let showCopiedAlert: boolean = false;
  let showAddParticipantModal: boolean = false;
  let newParticipantName: string = '';

  // Computed properties
  $: filteredParticipants = participants
    .filter(participant => {
      if (showCheckedInList) {
        return participant.checkedIn && 
          (searchText === '' || participant.name.toLowerCase().includes(searchText.toLowerCase()));
      } else {
        return !participant.checkedIn && 
          (searchText === '' || participant.name.toLowerCase().includes(searchText.toLowerCase()));
      }
    })
    .sort((a, b) => a.name.toLowerCase().localeCompare(b.name.toLowerCase()));

  $: checkedInCount = participants.filter(p => p.checkedIn).length;

  // Toggle participant check-in status
  function toggleCheckIn(id: string): void {
    participants = participants.map(p => 
      p.id === id ? { ...p, checkedIn: !p.checkedIn } : p
    );
  }

  // Import participants from text
  function importParticipants(): void {
    const lines = importText.split('\n');
    const newParticipants: Participant[] = lines
      .map(line => line.trim())
      .filter(line => line !== '')
      .map(name => ({ 
        id: Math.random().toString(36).substr(2, 9),
        name,
        checkedIn: false 
      }));
    
    participants = newParticipants;
    closeImportModal();
  }

  // Add a single participant
  function addSingleParticipant(): void {
    if (newParticipantName.trim() === '') return;
    
    const newParticipant: Participant = {
      id: Math.random().toString(36).substr(2, 9),
      name: newParticipantName.trim(),
      checkedIn: false
    };
    
    participants = [...participants, newParticipant];
    newParticipantName = '';
    showAddParticipantModal = false;
  }

  // Import from clipboard
  async function importFromClipboard(): Promise<void> {
    try {
      const text = await navigator.clipboard.readText();
      importText = text;
      importParticipants();
    } catch (err) {
      console.error('Failed to read clipboard: ', err);
      showTextInput = true;
    }
  }

  // Copy checked-in participants to clipboard
  function copyCheckedInToClipboard(): void {
    const checkedInList = participants
      .filter(p => p.checkedIn)
      .map(p => p.name)
      .join('\n');
    
    navigator.clipboard.writeText(checkedInList);
    showCopiedAlert = true;
    setTimeout(() => showCopiedAlert = false, 2000);
  }

  // Check/uncheck all
  function checkAll(): void {
    participants = participants.map(p => 
      !p.checkedIn ? { ...p, checkedIn: true } : p
    );
  }

  function uncheckAll(): void {
    participants = participants.map(p => 
      p.checkedIn ? { ...p, checkedIn: false } : p
    );
  }

  // Close import modal
  function closeImportModal(): void {
    showImportModal = false;
    showTextInput = false;
    importText = '';
  }

  // Clear search text
  function clearSearch(): void {
    searchText = '';
  }

  // Load data from localStorage on mount
  onMount(() => {
    const savedParticipants = localStorage.getItem('tournament-participants');
    if (savedParticipants) {
      participants = JSON.parse(savedParticipants);
    }
  });
  
  // Save data whenever participants change
  $: {
    if (participants.length > 0) {
      localStorage.setItem('tournament-participants', JSON.stringify(participants));
    }
  }
</script>

<div class="flex flex-col h-screen bg-gray-50">
  <!-- Header -->
  <header class="bg-white shadow-sm p-4">
    <div class="container mx-auto">
      <div class="flex justify-between items-center">
        <h1 class="text-xl font-bold text-gray-800">Tournament Check-in Tool</h1>
        <span class="text-sm text-gray-600">
          {checkedInCount}/{participants.length} Checked In
        </span>
      </div>
    </div>
  </header>

  <!-- Main Content -->
  <main class="flex-1 container mx-auto p-4 overflow-y-auto">
    {#if participants.length > 0}
      <!-- Search Bar -->
      <div class="mb-6 mt-2">
        <div class="relative">
          <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
            <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
            </svg>
          </div>
          <input
            type="text"
            class="block w-full p-2 pl-10 text-sm border border-gray-300 rounded-lg bg-white"
            placeholder="Search participants..."
            bind:value={searchText}
          />
          {#if searchText}
            <button 
              class="absolute inset-y-0 right-0 flex items-center pr-3"
              on:click={clearSearch}
            >
              <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          {/if}
        </div>
      </div>

      <!-- Add Participant Button (New) -->
      <div class="mb-4">
        <button
          on:click={() => showAddParticipantModal = true}
          class="w-full px-4 py-2 bg-green-600 text-white rounded-md flex items-center justify-center text-sm"
        >
          <svg class="h-5 w-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
          </svg>
          Add Participant
        </button>
      </div>

      <!-- Toggle View -->
      <div class="mb-6">
        <div class="flex rounded-md shadow-sm">
          <button
            class="flex-1 px-4 py-2 text-sm font-medium rounded-l-lg {!showCheckedInList 
              ? 'bg-blue-600 text-white' 
              : 'bg-white text-gray-700 border border-gray-300'}"
            on:click={() => showCheckedInList = false}
          >
            Not Checked In
          </button>
          <button
            class="flex-1 px-4 py-2 text-sm font-medium rounded-r-lg {showCheckedInList 
              ? 'bg-blue-600 text-white' 
              : 'bg-white text-gray-700 border border-gray-300'}"
            on:click={() => showCheckedInList = true}
          >
            Already Checked In
          </button>
        </div>
      </div>

      <!-- Participant List -->
      {#if filteredParticipants.length === 0}
        <div class="flex flex-col items-center justify-center py-12 text-center">
          {#if showCheckedInList}
            <div class="mb-4 p-4 bg-gray-100 rounded-full">
              <svg class="h-10 w-10 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
              </svg>
            </div>
            <h3 class="text-lg font-medium text-gray-900">No participants have been checked in yet</h3>
          {:else}
            <div class="mb-4 p-4 bg-green-100 rounded-full">
              <svg class="h-10 w-10 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
            </div>
            {#if searchText}
              <h3 class="text-lg font-medium text-gray-900">No participants match your search</h3>
            {:else}
              <h3 class="text-lg font-medium text-green-600">All participants have been checked in!</h3>
            {/if}
          {/if}
        </div>
      {:else}
        <ul class="divide-y divide-gray-200 border border-gray-200 rounded-lg bg-white shadow">
          {#each filteredParticipants as participant (participant.id)}
            <li class="flex justify-between items-center py-4 px-4">
              <span class="text-lg text-gray-800">{participant.name}</span>
              <button
                on:click={() => toggleCheckIn(participant.id)}
                class="p-2 rounded-full {
                  showCheckedInList 
                    ? 'text-red-500 hover:bg-red-50' 
                    : 'text-green-500 hover:bg-green-50'
                }"
              >
                {#if showCheckedInList}
                  <svg class="h-8 w-8" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                  </svg>
                {:else}
                  <svg class="h-8 w-8" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                  </svg>
                {/if}
              </button>
            </li>
          {/each}
        </ul>
      {/if}
    {:else}
      <div class="flex flex-col items-center justify-center h-full text-center">
        <div class="mb-6 p-6 bg-gray-100 rounded-full">
          <svg class="h-16 w-16 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z"></path>
          </svg>
        </div>
        <h2 class="text-xl font-medium text-gray-900 mb-2">No participants yet</h2>
        <p class="text-gray-500 mb-6">Import your participant list to get started</p>
        <div class="flex flex-col sm:flex-row gap-3">
          <button
            on:click={() => showImportModal = true}
            class="px-4 py-2 bg-blue-600 text-white rounded-md flex items-center justify-center"
          >
            <svg class="h-5 w-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"></path>
            </svg>
            Import Participants
          </button>
          <button
            on:click={() => showAddParticipantModal = true}
            class="px-4 py-2 bg-green-600 text-white rounded-md flex items-center justify-center"
          >
            <svg class="h-5 w-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
            </svg>
            Add Participant
          </button>
        </div>
      </div>
    {/if}
  </main>

  <!-- Footer -->
  {#if participants.length > 0}
    <footer class="bg-white border-t border-gray-200 p-4">
      <div class="container mx-auto flex justify-between items-center flex-wrap gap-2">
        <div class="flex gap-2">
          <button
            on:click={() => showImportModal = true}
            class="px-4 py-2 border border-gray-300 rounded-md flex items-center text-sm"
          >
            <svg class="h-4 w-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"></path>
            </svg>
            New Import
          </button>
        </div>
        
        <button
          on:click={copyCheckedInToClipboard}
          class="px-4 py-2 bg-blue-600 text-white rounded-md flex items-center text-sm"
        >
          <svg class="h-4 w-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"></path>
          </svg>
          Copy Checked-in
        </button>
        
        {#if showCheckedInList}
          <button
            on:click={uncheckAll}
            class="px-4 py-2 border border-gray-300 rounded-md flex items-center text-sm"
          >
            <svg class="h-4 w-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
            </svg>
            Uncheck All
          </button>
        {:else}
          <button
            on:click={checkAll}
            class="px-4 py-2 border border-gray-300 rounded-md flex items-center text-sm"
          >
            <svg class="h-4 w-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
            </svg>
            Check All
          </button>
        {/if}
      </div>
    </footer>
  {/if}

  <!-- Import Modal -->
  {#if showImportModal}
    <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-lg shadow-xl max-w-md w-full">
        {#if showTextInput}
          <div class="p-6">
            <h3 class="text-lg font-medium text-gray-900 mb-4">
              Paste your participant list below (one per line)
            </h3>
            <textarea
              class="w-full border border-gray-300 rounded-md p-2 h-48 mb-4"
              bind:value={importText}
              placeholder="John Doe&#10;Jane Smith&#10;..."
            ></textarea>
            <div class="flex justify-between">
              <button
                on:click={() => showTextInput = false}
                class="px-4 py-2 border border-gray-300 rounded-md"
              >
                Back
              </button>
              <button
                on:click={importParticipants}
                class="px-4 py-2 bg-blue-600 text-white rounded-md"
              >
                Import
              </button>
            </div>
          </div>
        {:else}
          <div class="p-6">
            <h3 class="text-xl font-medium text-gray-900 text-center mb-6">
              Import Participants
            </h3>
            <div class="space-y-4 mb-6">
              <button
                on:click={importFromClipboard}
                class="w-full flex items-center p-4 border border-gray-200 rounded-lg hover:bg-blue-50"
              >
                <div class="p-2 bg-blue-100 rounded-full">
                  <svg class="h-6 w-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"></path>
                  </svg>
                </div>
                <div class="ml-4 text-left">
                  <div class="text-sm font-medium text-gray-900">Import from Clipboard</div>
                  <div class="text-sm text-gray-500">Use list already copied to clipboard</div>
                </div>
              </button>
              
              <button
                on:click={() => showTextInput = true}
                class="w-full flex items-center p-4 border border-gray-200 rounded-lg hover:bg-green-50"
              >
                <div class="p-2 bg-green-100 rounded-full">
                  <svg class="h-6 w-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path>
                  </svg>
                </div>
                <div class="ml-4 text-left">
                  <div class="text-sm font-medium text-gray-900">Enter/Paste Manually</div>
                  <div class="text-sm text-gray-500">Type or paste participant list</div>
                </div>
              </button>
            </div>
            <div class="flex justify-end">
              <button
                on:click={closeImportModal}
                class="px-4 py-2 border border-gray-300 rounded-md"
              >
                Cancel
              </button>
            </div>
          </div>
        {/if}
      </div>
    </div>
  {/if}

  <!-- Add Participant Modal (New) -->
  {#if showAddParticipantModal}
    <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-lg shadow-xl max-w-md w-full">
        <div class="p-6">
          <h3 class="text-lg font-medium text-gray-900 mb-4">
            Add New Participant
          </h3>
          <form on:submit|preventDefault={addSingleParticipant}>
            <div class="mb-4">
              <label for="participantName" class="block text-sm font-medium text-gray-700 mb-1">
                Participant Name
              </label>
              <input
                id="participantName"
                type="text"
                class="w-full border border-gray-300 rounded-md p-2"
                placeholder="Enter participant name"
                bind:value={newParticipantName}
                required
                autofocus
              />
            </div>
            <div class="flex justify-end space-x-3">
              <button
                type="button"
                on:click={() => {
                  showAddParticipantModal = false;
                  newParticipantName = '';
                }}
                class="px-4 py-2 border border-gray-300 rounded-md"
              >
                Cancel
              </button>
              <button
                type="submit"
                class="px-4 py-2 bg-green-600 text-white rounded-md"
                disabled={!newParticipantName.trim()}
              >
                Add
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  {/if}

  <!-- Copied Alert -->
  {#if showCopiedAlert}
    <div class="fixed bottom-4 right-4 bg-green-600 text-white px-4 py-2 rounded-md shadow-lg z-50">
      Participant list copied to clipboard!
    </div>
  {/if}
</div>