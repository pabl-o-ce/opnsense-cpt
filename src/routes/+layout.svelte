<script lang="ts">
	import '../app.css';
	let { children } = $props();
    export const prerender = true;
    export const ssr = false;
</script>

<div class="min-h-screen bg-zinc-50 dark:bg-zinc-950 flex flex-col">
    <header class="bg-zinc-100 border-b border-gray-200">
      <nav class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="h-16 flex items-center">
          <img src="favicon.png" alt="Logo" class="h-8 w-auto" />
        </div>
      </nav>
    </header>
{@render children()}
</div>
