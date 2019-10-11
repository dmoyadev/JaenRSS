<template>
	<article class="article">
		<div class="content">
			<details>
				<summary>
					<picture>
						<img v-if="article.enclosure && article.enclosure.url && article.enclosure.width && article.enclosure.height"
						     :src="article.enclosure.url" :width="article.enclosure.width" :height="article.enclosure.height" alt="imagen del artículo">
						<img v-else-if="article.imagen"
						     :src="article.imagen" width="320px" height="180px" alt="imagen del artículo">
						<img v-else src="https://via.placeholder.com/320x180?text=No hay imagen" width="320px" height="180px" alt="placeholder">
					</picture>
					<header>
						<span class="date dynamic-text">({{ index }}) {{ article.pubDate }} - <a :href="article.link" target="_blank">Enlace</a></span>
						<h1 class="dynamic-text">{{ article.title }}</h1>
						<span class="summary dynamic-text">{{ getPreview(article.contentSnippet) }}...</span>
					</header>
				</summary>
				<p class="text dynamic-text" v-html="parseContent(article.content)"></p>
			</details>
		</div>
	</article>
</template>

<script>
	export default {
		name: "Article",
		props: {
			article: {Array, Object},
			index: Number
		},
		methods: {
			parseContent(content) {
				return content.substr(content.search("\">") + 9);
			},
			getPreview(content) {
				return content.substr(0, 120);
			},
		}
	}
</script>

<style lang="scss">
	.article {
		max-width: 40%;
		text-align: left;
		display: flex;
		align-items: flex-start;
		border-bottom: solid 1px gray;
		padding: 50px;

		@media (max-width: 800px) {
			flex-direction: column;
			max-width: 80%;
		}

		.content {
			display: flex;
			flex-direction: column;

			details {
				summary {
					display: flex;
					outline: none;
					cursor: pointer;

					img {
						margin-right: 20px;
						min-width: 320px;
						min-height: 180px;

						@media (max-width: 800px) {
							margin-bottom: 10px;
							min-width: 100%;
							min-height: auto;
						}
					}

					header {
						display: flex;
						flex-direction: column;

						h1 {
							padding-bottom: 10px;
							margin: 0;
						}

						.date {
							font-size: 12px;
							font-style: italic;
						}

						.summary {
							font-size: 14px;
							font-style: italic;
						}

					}

					&::-webkit-details-marker {
						display: none;
					}

					@media (max-width: 800px) {
						flex-direction: column;
					}

				}

				&[open] {
					img {
						margin-bottom: 10px;
					}

					.summary {
						display: none;
					}
				}
			}

			.text {
				text-align: justify;

				p {
					margin-bottom: 15px;
				}
			}
		}
	}
</style>
