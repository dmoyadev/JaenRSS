<template>
	<div class="home">
		<img class="logo" src="../assets/logo.svg" alt="logo de JaénRSS">
		<h4>A RSS reader by <a href="mailto:mail.danielml@gmail.com">Daniel Moya</a></h4>
		<nav>
			<form>
				<fieldset>
					<legend class="dynamic-text">Elegir otro RSS:</legend>
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
						<span>
							<input type="radio" id="DiarioDigitalUJA" value="https://diariodigital.ujaen.es/rss.xml" v-model="feedUrl"><label for="DiarioDigitalUJA">Diario Digital UJA</label>
						</span>
					</section>
					<section>
						<label class="dynamic-text">Leer RSS por su URL:<input class="input" type="url" v-model="feedUrl"></label>
					</section>
				</fieldset>
			</form>
			<form>
				<fieldset>
					<legend class="dynamic-text">Cambiar estilo:</legend>
					<section class="newspaper">
						<input class="text-button" type="button" value="- Disminuir tamaño de letra" @click="changeFontSize(-1)">
						<input class="text-button" type="button" value="+ Aumentar tamaño de letra" @click="changeFontSize(1)">
					</section>
					<section>
						<input class="text-button" type="button" value="Establecer tamaño por defecto" @click="changeFontSize(0)">
					</section>
				</fieldset>
			</form>
		</nav>
		<div v-if="error" class="error">{{error}}</div>
		<div v-else class="feed">
			<h2 class="dynamic-text">{{ feed.title }}</h2>
			<div v-if="loading" class="spinner">
				<div class="bounce1"></div>
				<div class="bounce2"></div>
				<div class="bounce3"></div>
			</div>
			<div class="articles-container">
				<Article v-for="(article, index) of getArticles()"
				         :article="article" :index="index"></Article>
			</div>
		</div>
	</div>
</template>

<script>
	import RSSParser from "rss-parser";
	import Article from "@/components/Article";

	const CORS_PROXY = "https://cors-anywhere.herokuapp.com/"
	export default {
		name: 'home',
		components: {
			Article
		},
		data() {
			return {
				feedUrl: "https://www.diariojaen.es/rss/jaen.xml",
				limit: 20,
				fontSizeDeviation: 0,
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
					const data = await fetch(CORS_PROXY + this.feedUrl);
					if (data.ok) {
						const text = await data.text();
						const parser = new RSSParser({
							customFields: {
								item: ['imagen']
							}
						});
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
			increaseLimit(loadMore = 5) {
				this.limit += loadMore;
			},
			changeFontSize(delta) {
				let txt = document.getElementsByClassName('dynamic-text');
				for (let element in txt) {
					if (typeof HTMLElement === "object" ? txt[element] instanceof HTMLElement : //DOM2
						txt[element] && typeof txt[element] === "object" && txt[element] !== null && txt[element].nodeType === 1 && typeof txt[element].nodeName === "string") {
						let style = window.getComputedStyle(txt[element], null).getPropertyValue('font-size');
						let currentSize = parseFloat(style);
						txt[element].style.fontSize =
							(delta !== 0)   ? (currentSize + delta) + 'px'
											: (currentSize - this.fontSizeDeviation) + 'px';
					}
				}
				this.fontSizeDeviation = (delta !== 0) ? this.fontSizeDeviation + delta : 0
			}
		}
	}
</script>

<style lang="scss">
	.home {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;

		.logo {
			margin-left: 40px;
			margin-top: 20px;
			width: 320px;
			height: auto;
		}

		h4 {
			color: #FF5D00;
			padding:20px;
		}

		h5 {
			padding: 20px;
		}

		nav {
			width: 80%;
			display: flex;
			justify-content: space-between;
			align-items: center;

			@media (max-width: 800px) {
				flex-direction: column;
			}

			form {
				width: 100%;
				padding: 0 50px 70px;

				@media (max-width: 800px) {
					padding-bottom: 30px;
				}

				fieldset {
					padding: 20px;
					border: 2px solid #FF5D00;
					box-shadow: 0 18px 18px rgba(0, 0, 0, 0.25), 0 5px 7px rgba(0, 0, 0, 0.22);

					legend {
						padding: 10px;
					}

					.newspaper {
						display: flex;
						justify-content: space-around;
						align-items: center;
						margin-bottom: 20px;

						span {
							display: flex;
							justify-content: center;
							align-items: center;

							input {
								margin-right: 10px;
							}

							label {
								margin-top: 2px;
							}
						}
					}

					.text-button {
						padding: 10px;
						border: 1px solid gray;
						outline: none;
						margin-right: 10px;
					}
				}

				.input {
					width: 90%;
					padding: 10px;
				}
			}
		}

		.feed {

			.articles-container {
				display: flex;
				flex-wrap: wrap;
				justify-content: center;
				align-items: flex-start;
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
