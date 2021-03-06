<template>
  <div class="container fill">
    <div v-if="fetching" class="spacer">
      <b-loading :is-full-page="false" :active="fetching"></b-loading>
    </div>

    <div v-if="!fetching">
      <header class="has-text-left">
        <h2 class="title is-3">
          {{ topic.title }}
        </h2>

        <CategoryTag :categoryId="topic.categoryId">
        </CategoryTag>
        &nbsp;
        <a class="topic-nav topic-nav-to-end" @click="scrollTo('endOfTopic')">
          Jump to end
        </a>
      </header>

      <br>

      <main ref="posts">
        <Post :data="topic"></Post>

        <Post v-for="(reply, index) in topic.replies"
          :data="reply"
          :isReply="true"
          :key="index"
        >
        </Post>
        <a class="topic-nav topic-nav-to-top" @click="scrollTo('topOfPage')">
          Back to Top
        </a>
      </main>

      <br>
      <a id="endOfTopic" />
      <ShowIfLoggedIn>
        <ReplyForm
          :fetching="$store.state.replies.fetching"
          :text="replyText"
          @input="onReplyInput"
          @submit="onReplySubmit">
        </ReplyForm>
      </ShowIfLoggedIn>
    </div>

  </div>
</template>

<script>
import { mapState } from 'vuex';

import Post from '@/components/Post.vue';
import ReplyForm from '@/components/ReplyForm.vue';
import ShowIfLoggedIn from '@/components/ShowIfLoggedIn.vue';
import CategoryTag from '@/components/CategoryTag.vue';
import { getTopic } from '../services/post.service.js';
import { errorAlertOptions } from '../utils/notifications.js';

export default {
  name: 'topic',
  components: {
    Post,
    ReplyForm,
    ShowIfLoggedIn,
    CategoryTag,
  },
  created() {
    this.fetchTopic();
  },
  data() {
    return {
      fetching: true,
      topic: {},
      replyText: '',
    };
  },
  computed: {
    ...mapState( 'categories', [
      'categoriesBySlug',
    ] ),
  },
  methods: {
    onReplyInput( text ) {
      this.replyText = text;
    },
    onReplySubmit() {
      const payload = {
        parentComment: this.topic,
        content: this.replyText,
      };

      this.$store.dispatch( 'replies/submitReply', payload )
        .then( ( reply ) => {
          if ( reply ) {
            this.fetchTopic();
            this.replyText = '';
          }
        } )
        .catch( ( err ) => {
          console.log( err );
          this.$toast.open( errorAlertOptions( 'Oops! Could not submit reply at this moment', err ) );
          this.$ga.exception( err );
        } );
    },
    fetchTopic() {
      const { author, permlink } = this.$route.params;

      this.fetching = true;

      return getTopic( author, permlink ).then( ( topic ) => {
        if ( !topic ) {
          return this.$router.push( '/' );
        }

        this.topic = topic;
        this.fetching = false;
      } );
    },
    categoryFromId( id ) {
      return ( this.categoriesBySlug[id] || { name: '' } ).name;
    },
    scrollTo( id ) {
      window.scrollTo( 0, document.getElementById( id ).offsetTop );
    },
  },
};
</script>
