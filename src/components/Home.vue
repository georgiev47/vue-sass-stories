<template>
  <div id="app">
    <h1>Hacker News Top Stories</h1>
    <div class="story-container">
        <div v-for="story in sortAscending" :key="story.storyTitle" class="story-box">
            <img alt="Vue logo" src="../assets/tell-me-a-story.jpg">
            <div class="story-text">
                <div>
                    <div class="score">Score: {{ story.storyScore }}</div>
                    <h3>{{ story.storyTitle }}</h3>
                </div>
                <div class="story-details">
                    <p><span class="bold">URL:</span> <a :href="story.storyUrl">{{ story.storyUrl }}</a></p>
                    <p><span class="bold">Story Timestamp:</span> {{ story.storyTimestamp }}</p>
                    <p><span class="bold">Author ID:</span> {{ story.storyAuthorID }}</p>
                    <p><span class="bold">Author Karma Score:</span> {{ story.storyAuthorKarma }}</p>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Home',
  components: {},
  mounted(){
      this.loadStories();
  },
  methods: {
      async loadStories(){
        let responseStoryID = await axios.get('https://hacker-news.firebaseio.com/v0/topstories.json');
        const { data } = responseStoryID;

        // this is not completely random, will need better algorithm like Fisher-Yates for perfect randomness
        const numberOfRandomElements = 10;
        let random = data.sort(() => 0.5 - Math.random()).slice(0,numberOfRandomElements);
        
        await Promise.all(random.map(item =>
            axios.get(`https://hacker-news.firebaseio.com/v0/item/${item}.json`)
            .then(resStory => {
                axios.get(`https://hacker-news.firebaseio.com/v0/user/${resStory.data.by}.json`)
                .then(resAuthor => {
                    this.stories.push({
                        storyTitle: resStory.data.title,
                        storyUrl: resStory.data.url,
                        storyTimestamp: resStory.data.time,
                        storyScore: resStory.data.score,
                        storyAuthorID: resStory.data.by,
                        storyAuthorKarma: resAuthor.data.karma
                    });
                })
            })
        ));
      }
  },
  computed: {
    sortAscending(){
        return this.stories.slice().sort((a, b) => a.storyScore - b.storyScore );
    }
  },
  data() {
      return {
        stories: []
      }
  }
}
</script>

<style lang="scss">
$primary-color: #2c3e50;
$secondary-color: white;

body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: $primary-color;
}

h1 {
    text-align: center;
}

.story-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;

    .story-box {
        background-color: $secondary-color;
        border-radius: 5px;
        margin: 10px;
        width: 22rem;
        box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.15), 0 6px 20px 0 rgba(0, 0, 0, 0.15);
        transition: all .2s ease-in-out;

        img{
            width: 100%;
            border-radius: 5px 5px 0 0;
        }

        .story-text {
            padding: 10px;

            div {
                padding: 0;
                text-align: center;

                &:nth-child(1){
                }

                .score {
                    padding: 5px 12px;
                    font-weight: bold;
                    display: inline-block;
                    background-color: $primary-color;
                    color: $secondary-color;
                    border-radius: 4px;
                }

                h3 {
                    font-weight: bold;
                    
                }
            }

            .story-details {
                text-align: left;

                .bold {
                    font-weight: bold;
                }

                p {
                    a {
                        display: inline-block;
                        text-overflow: ellipsis;
                        max-width: 18em;
                        white-space: nowrap;
                        overflow: hidden;
                        vertical-align: middle;
                    }
                }
            }
            
        }

        &:hover {
            box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.35), 0 6px 20px 0 rgba(0, 0, 0, 0.35);
        }

        @media (max-width: 768px) {
            &:active {
                transform: scale(1.03);
                transition: 0.1s;
            }
        }
        
    }
}
</style>