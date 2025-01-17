<!--
 KalawatiPanel - Management App for kalawatienterprises.co.in
 Copyright (C) 2022  Vidhu Kant Sharma <vidhukant@protonmail.ch>

 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.

 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.

 You should have received a copy of the GNU General Public License
 along with this program.  If not, see <https://www.gnu.org/licenses/>.
-->

<script>
  import { fade } from 'svelte/transition';
  import CategoryEditor from "../components/CategoryEditor.svelte";
  import { Category } from "../classes";

  const filterCategories = (categories, searchString) => [...new Set([
    ...categories.filter(i => i.Name.toLowerCase().includes(searchString.toLowerCase())),
    ...categories.filter(i => (i.Parent != null) && i.Parent.Name.toLowerCase().includes(searchString.toLowerCase()))
  ])];

  let categories = [];
  const loadCategories = () =>
    fetch('http://localhost:4001/api/categories')
      .then(response => response.json())
      .then(data => categories = data);
  loadCategories();

  let searchString = "";
  let showEditor = false;
  let editorCategory = new Category();

  const handleAddNew = () => {
    showEditor = true;
    const c = new Category();
    c.Parent = new Category();
    editorCategory = c;
  }

  const handleCancel = () => {
    showEditor = false;
    editorCategory = new Category();
  }

  const handleUpdate = () => {
    loadCategories();
    handleCancel();
  }
</script>

<div class="searchbox">
  Search Categories <input bind:value={searchString}/>
</div>

{#key categories}
<table in:fade="{{ duration: 200, delay: 100 }}" out:fade="{{ duration: 300, delay: 0 }}">
  <tr>
    <th>Name</th>
    <th>Parent</th>
    <button on:click={handleAddNew}>Add New</button>
  </tr>
  {#if categories != null}
    {#each filterCategories(categories, searchString) as c}
      <tr>
        <td>{c.Name}</td>
        <td>{c.Parent == null ? "" : c.Parent.Name}</td>
        <button on:click={() => {
          // Prevents table from reloading while also does the job
          const categoryToEdit = c;
          categoryToEdit.Parent = c.Parent == null ? new Category() : c.Parent;
          editorCategory = categoryToEdit;
          showEditor = true;
        }}>
          Edit
        </button>
      </tr>
    {:else}
      <h1>No Categories With This Name Found.</h1>
    {/each}
  {:else}
    <h1>No Categories Exist In The Database.</h1>
  {/if}
</table>
{/key}

{#if showEditor}
<div class="editor-parent" in:fade="{{ duration: 150 }}" out:fade="{{ duration: 200 }}">
  <CategoryEditor category={editorCategory} on:categories-updated={handleUpdate} on:edit-canceled={handleCancel}/>
</div>
{/if}

<style>
  button {
    width: 7rem;
  }
  .searchbox {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 2rem 0;
  }
  .searchbox input {
    margin-left: 1rem;
  }
  table {
    width: 30%;
    margin: auto;
  }
  th, td {
    text-align: left;
    border-bottom: 1px solid black;
    width: 100%;
  }
  .editor-parent {
    z-index: 10;
    background-color: white;
    position: fixed;
    top: 0; bottom: 0;
    left: 0; right: 0;
    margin: auto;
    border: 1px solid black;
    height: 40vh;
    width: 60vw;
    box-shadow: 0.1rem 0.1rem 0.3rem 0.05rem rgba(35, 38, 39, 0.7);
  }
</style>
