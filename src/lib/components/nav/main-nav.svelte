<script lang="ts">
	import { page } from '$app/stores';
	import { siteConfig } from '$lib/config/site';
	import { cn } from '$lib/utils';
	import { Icons } from '$lib/icons';
</script>

<div class="mr-4 hidden md:flex">
	<a href="/" class="mr-6 flex items-center space-x-2">
		<Icons.logo class="h-6 w-6" />
		<span class="hidden text-[15px] font-bold sm:inline-block lg:text-base">
			{siteConfig.name}
		</span>
	</a>
	<nav class="flex items-center space-x-6 text-sm font-medium">
		<a
			href="/demo"
			class={cn(
				'transition-colors hover:text-foreground/80',
				$page.url.pathname.startsWith('/docs/components') ? 'text-foreground' : 'text-foreground/60'
			)}
		>
			Demonstration
		</a>
		<a
			href="/contact"
			class={cn(
				'transition-colors hover:text-foreground/80',
				$page.url.pathname.startsWith('/themes') ? 'text-foreground' : 'text-foreground/60'
			)}
		>
			Contact Us
		</a>
		<a
			href="https://dual-gan-anonymizer.vercel.app"
			class={cn(
				'transition-colors hover:text-foreground/80',
				$page.url.pathname === '/docs' ? 'text-foreground' : 'text-foreground/60'
			)}
		>
			Documentation
		</a>
		<a
			href={siteConfig.links.github}
			target="_blank"
			rel="noopener noreferrer"
			class={cn('hidden text-foreground/60 transition-colors hover:text-foreground/80 xl:block')}
		>
			GitHub
		</a>
	</nav>
</div>
