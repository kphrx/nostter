<script lang="ts" context="module">
	declare global {
		interface Window {
			twttr: any;
		}
	}
</script>

<script lang="ts">
	import { _ } from 'svelte-i18n';
	import { newUrl } from '$lib/Helper';
	import { enablePreview } from '$lib/stores/Preference';
	import Text from './Text.svelte';
	import ExternalLink from '$lib/components/ExternalLink.svelte';
	import YouTube from '$lib/components/content/YouTube.svelte';
	import Img from '$lib/components/content/Img.svelte';

	export let text: string;
	export let url: string | undefined = undefined;

	$: link = newUrl(url ?? text);

	let twitterWidget: HTMLDivElement | undefined;

	$: if (twitterWidget !== undefined) {
		console.debug('[twitter]', twitterWidget);
		window.twttr?.widgets.load(twitterWidget);
	}

	let preview = $enablePreview;
</script>

{#if link === undefined}
	<Text {text} />
{:else if link.origin === 'https://twitter.com' || link.origin === 'https://x.com'}
	{#if preview}
		<div bind:this={twitterWidget}>
			<blockquote class="twitter-tweet">
				<a
					href={link.href.replace('x.com', 'twitter.com')}
					target="_blank"
					rel="noopener noreferrer"
				>
					{link.href}
				</a>
			</blockquote>
		</div>
	{:else}
		<ExternalLink {link} />
		<button on:click={() => (preview = true)}>{$_('content.show')}</button>
	{/if}
{:else if link.hostname === 'youtu.be' || /^(.+\.)*youtube\.com$/s.test(link.hostname)}
	<YouTube {link} />
{:else if link.hostname === 'amzn.to' || link.hostname === 'amzn.asia' || /^(.+\.)*amazon\.co\.jp$/s.test(link.hostname)}
	<ExternalLink {link} />
{:else if /\.(apng|avif|gif|jpg|jpeg|png|webp|bmp)$/i.test(link.pathname)}
	{#if preview}
		<div>
			<a href={link.href} target="_blank" rel="noopener noreferrer">
				<Img url={link} />
			</a>
		</div>
	{:else}
		<ExternalLink {link} />
		<button on:click={() => (preview = true)}>{$_('content.show')}</button>
	{/if}
{:else if /\.(mp3|m4a|wav)$/i.test(link.pathname)}
	{#if preview}
		<audio src={link.href} controls />
	{:else}
		<ExternalLink {link} />
		<button on:click={() => (preview = true)}>{$_('content.show')}</button>
	{/if}
{:else if /\.(mp4|ogg|webm|ogv|mov|mkv|avi|m4v)$/i.test(link.pathname)}
	{#if preview}
		<!-- svelte-ignore a11y-media-has-caption -->
		<div>
			<video src={link.href} controls />
		</div>
	{:else}
		<ExternalLink {link} />
		<button on:click={() => (preview = true)}>{$_('content.show')}</button>
	{/if}
{:else if url !== undefined && text !== url}
	<ExternalLink {link}>{text}</ExternalLink>
{:else}
	<ExternalLink {link} />
{/if}

<style>
	video {
		max-width: calc(100% - 1.5em);
		max-height: 20em;
		margin: 0.5em;
	}
</style>
