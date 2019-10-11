<template>
	<div class="home">
		<h1>JaénRSS</h1>
		<h4>A RSS reader by <a href="mailto:mail.danielml@gmail.com">Daniel Moya</a></h4>
		<form>
			<fieldset>
				<legend>Elegir otro RSS:</legend>
				<section class="newspaper">
					<span>
						<input type="radio" id="DiarioJaen" value="https://www.diariojaen.es/rss/jaen.xml" v-model="feedUrl"><label for="DiarioJaen">Diario Jaén</label>
					</span>
					<span>
						<input type="radio" id="IdealJaen" value="https://www.ideal.es/rss/2.0/?section=jaen/jaen" v-model="feedUrl"><label for="IdealJaen">Ideal Jaén</label>
					</span>
					<span>
						<input type="radio" id="20MinutosJaen" value="https://www.20minutos.es/rss/jaen/" v-model="feedUrl"><label for="20MinutosJaen">20 Minutos Jaén</label>
					</span>
				</section>
				<section>
					<label>Leer RSS por su URL<input class="input" type="url" v-model="feedUrl"></label>
				</section>
			</fieldset>
		</form>
		<div v-if="error" class="error">{{error}}</div>
		<div v-else class="feed">
			<h2>{{ feed.title }}</h2>
			<div v-if="loading" class="spinner">
				<div class="bounce1"></div>
				<div class="bounce2"></div>
				<div class="bounce3"></div>
			</div>
			<div class="articles-container">
				<article v-for="(article, index) of getArticles()" class="article">
					<img v-if="article.enclosure && article.enclosure.url && article.enclosure.width && article.enclosure.height"
					     :src="article.enclosure.url" :width="article.enclosure.width" :height="article.enclosure.height" alt="imagen del artículo">
					<img v-else-if="article.imagen"
					     :src="article.imagen" alt="imagen del artículo">
					<img v-else src="https://via.placeholder.com/320x180?text=No hay imagen" width="320px" height="180px" alt="placeholder">
					<div class="content">
						<details>
							<summary>
								<span class="date">({{ index }}) {{ article.pubDate }} - <a :href="article.link" target="_blank">Enlace</a></span>
								<h1>{{ article.title }}</h1>
								<span class="summary">{{ getPreview(article.contentSnippet) }}</span>
							</summary>
							<p v-html="parseContent(article.content)"></p>
						</details>
					</div>
				</article>
			</div>
		</div>
	</div>
</template>

<script>
	import RSSParser from "rss-parser";

	export default {
		name: 'home',
		components: {
		},
		data() {
			return {
				feedUrl: "https://www.diariojaen.es/rss/jaen.xml",
				limit: 20,
				loadMore: true,
				loading: true,
				error: "",
				feed: {}
			};
		},
		created() {
			this.fetchData();
		},
		mounted() {
			this.scrollEvent = window.addEventListener("scroll", () => {
				if (
					window.innerHeight + window.scrollY >=
					document.body.offsetHeight - 200
				) {
					this.increaseLimit();
				}
			});
		},
		destroyed() {
			if (this.scrollEvent) {
				window.removeEventListener(this.scrollEvent);
			}
		},
		watch: {
			feedUrl() {
				this.fetchData();
			},
			limit(newVal, oldVal) {
				if (newVal !== oldVal) {
					this.getArticles();
				}
			}
		},
		methods: {
			async fetchData() {
				this.error = "";
				this.loading = true;
				this.feed = {};
				try {
					const data = await fetch(this.feedUrl);
					if (data.ok) {
						const text = await data.text();
						const parser = new RSSParser();
						parser.parseString(text, (err, parsed) => {
							this.loading = false;
							if (err) {
								this.error = `Error occured while parsing RSS Feed ${err.toString()}`;
							} else {
								this.feed = parsed;
							}
						});
					} else {
						this.error = "Error occured while fetching the feed";
						this.loading = false;
					}
				} catch (e) {
					if (e.toString() === "TypeError: Failed to fetch") {
						this.error = "Error due to CORS policy";
					} else {
						this.error = e.toString();
					}
					this.loading = false;
				}
			},
			getArticles() {
				if (this.feed.items && this.feed.items) {
					return this.feed.items.slice(0, this.limit);
				}
			},
			parseContent(content) {
				return content.substr(content.search("\">") + 9);
			},
			getPreview(content) {
				return content.substr(0, 300);
			},
			increaseLimit(loadMore = 5) {
				this.limit += loadMore;
			},
		}
	}
</script>

<style lang="scss">
	.home {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;

		form {
			width: 80%;
			.newspaper {
				display: flex;
				justify-content: space-around;
				align-items: center;
				margin-bottom: 20px;
			}

			.input {
				width: 90%;
				padding: 10px;
			}
		}

		.feed {

			.articles-container {
				max-width: 1000px;

				.article {
					text-align: left;
					display: flex;
					align-items: flex-start;
					border-bottom: solid 1px gray;
					padding: 20px;

					@media (max-width: 800px) {
						flex-direction: column;
					}

					img {
						margin-right: 20px;
						min-width: 320px;
						min-height: 180px;

						@media (max-width: 800px) {
							min-width: 100%;
							min-height: auto;
							margin-bottom: 10px;
						}
					}

					.content {
						display: flex;
						flex-direction: column;

						summary {
							display: flex;
							flex-direction: column;
							outline: none;
							text-align: justify;
							cursor: pointer;

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

							&::-webkit-details-marker {
								display: none;
							}

							&[open] {
								.summary {
									display: none;
								}
							}
						}


					}
				}
			}

			/* CSS Spinner */
			.spinner {
				margin: 40px auto 0;
				width: 150px;
				text-align: center;
			}

			.error {
				color: red;
			}

			.spinner > div {
				width: 18px;
				height: 18px;
				/* background-color: #ff641b; */
				background-color: #42b983;
				background-color: #777;
				margin-right: 10px;
				border-radius: 100%;
				display: inline-block;
				-webkit-animation: sk-bouncedelay 1.4s infinite ease-in-out both;
				animation: sk-bouncedelay 1.4s infinite ease-in-out both;
			}

			.spinner .bounce1 {
				-webkit-animation-delay: -0.32s;
				animation-delay: -0.32s;
			}

			.spinner .bounce2 {
				-webkit-animation-delay: -0.16s;
				animation-delay: -0.16s;
			}

			@-webkit-keyframes sk-bouncedelay {
				0%,
				80%,
				100% {
					-webkit-transform: scale(0);
				}
				40% {
					-webkit-transform: scale(1);
				}
			}
			@keyframes sk-bouncedelay {
				0%,
				80%,
				100% {
					-webkit-transform: scale(0);
					transform: scale(0);
				}
				40% {
					-webkit-transform: scale(1);
					transform: scale(1);
				}
			}
		}
	}
</style>
