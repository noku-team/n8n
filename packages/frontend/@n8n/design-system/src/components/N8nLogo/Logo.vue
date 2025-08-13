<script setup lang="ts">
import type { FrontendSettings } from '@n8n/api-types';
import { useFavicon } from '@vueuse/core';
import { computed, onMounted, useCssModule, useTemplateRef } from 'vue';
import LogoCollapsed from './logo-icon.svg';
import LogoExpanded from './original-full.svg';
//
const props = defineProps<
	(
		| {
				size: 'large';
		  }
		| {
				size: 'small';
				collapsed: boolean;
		  }
	) & {
		releaseChannel?: 'stable' | 'beta' | 'nightly' | 'dev' | 'rc';
	}
>();

const { size, releaseChannel } = props;

// const showLogoText = computed(() => {
// 	if (location === 'authView') return true;
// 	return !props.collapsed;
// });

const currentLogo = computed(() => {
	if (location === 'authView') return LogoExpanded;
	return 'collapsed' in props && props.collapsed ? LogoCollapsed : LogoExpanded;
});

const $style = useCssModule();
const containerClasses = computed(() => {
	if (size === 'large') {
		return [$style.logoContainer, $style.large];
	}
	return [
		$style.logoContainer,
		$style.sidebar,
		props.collapsed ? $style.sidebarCollapsed : $style.sidebarExpanded,
	];
});

const svg = useTemplateRef<{ $el: Element }>('logo');
onMounted(() => {
	if (!releaseChannel || releaseChannel === 'stable' || !('createObjectURL' in URL)) {
		return;
	}

	const logoEl = svg.value!.$el;

	// Change the logo fill color inline, so that favicon can also use it
	const logoColor = releaseChannel === 'dev' ? '#838383' : '#E9984B';
	logoEl.querySelector('path')?.setAttribute('fill', logoColor);

	// Reuse the SVG as favicon
	const blob = new Blob([logoEl.outerHTML], { type: 'image/svg+xml' });
	useFavicon(URL.createObjectURL(blob));
});
</script>

<template>
	<div :class="containerClasses" data-test-id="n8n-logo">
		<component :is="currentLogo" ref="logo" :class="$style.logo" />
		<!-- <LogoText v-if="showLogoText" :class="$style.logoText" /> -->
		<slot />
	</div>
</template>

<style lang="scss" module>
.logoContainer {
	display: flex;
	justify-content: center;
	align-items: center;
}

.logoText {
	margin-left: var(--spacing--5xs);
	path {
		fill: var(--color--text--shade-1);
	}
}

.large {
	transform: scale(2);
	margin-bottom: var(--spacing--xl);

	.logo,
	.logoText {
		// transform: scale(0.4) translateY(-2px);
		height: 40px;
		width: auto;
	}

	.logoText {
		margin-left: var(--spacing--xs);
		margin-right: var(--spacing--3xs);
	}
}

.sidebarExpanded .logo {
	margin-left: var(--spacing--2xs);
	width: 100px;
	height: auto;
}

.sidebarCollapsed .logo {
	width: 40px;
	height: 40px;
}
</style>
