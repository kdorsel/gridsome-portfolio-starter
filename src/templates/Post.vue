<template>
  <Layout>
    <div class="container mx-auto my-16">
      <h1 class="text-4xl font-bold leading-tight">{{ $page.post.title }}</h1>
      <div class="text-xl text-gray-600 mb-3">{{ $page.post.date }}</div>
      <div class="flex flex-wrap mb-3 text-xs">
        <g-link
          v-for="tag in $page.post.tags"
          :key="tag.id"
          :to="tag.path"
          class="bg-gray-300 rounded-full px-4 py-2 mr-2 mb-2 hover:bg-green-300"
        >
          {{ tag.title }}
        </g-link>
      </div>
      <!-- eslint-disable-next-line vue/no-v-html -->
      <div class="markdown-body mb-8" v-html="$page.post.content" />
      <iframe v-if="$page.post.form.length > 0" :src="$page.post.form" width="640" height="802" frameborder="0" marginheight="0" marginwidth="0">Loading…</iframe>
      <div class="mb-8">
        <g-link to="/blog" class="font-bold uppercase">Back to Blog</g-link>
      </div>
      <div id="graphcomment"></div>
    </div>
  </Layout>
</template>

<page-query>
query Post($path: String!) {
  post: post(path: $path) {
    title
    date(format: "MMMM D, Y")
    form
    content
    tags {
      title
      path
    }
  }
}
</page-query>

<script>
export default {
  metaInfo() {
    return {
      title: this.$page.post.title,
      meta: [
        {
          name: "description",
          content: this.$page.post.summary,
        },
        {
          property: "og:title",
          content: this.$page.post.title,
        },
        {
          property: "og:url",
          cotent: this.$page.post.path,
        },
        {
          property: "og:description",
          cotent: this.$page.post.summary,
        },
        {
          property: "og:type",
          cotent: "article",
        },
        {
          property: "og:article:published_time",
          cotent: this.$page.post.date,
        },
      ],
    };
  },
  mounted() {
    window.gc_params = {
      graphcomment_id: "kassymdorsel",
      // if your website has a fixed header, indicate it's height in pixels
      fixed_header_height: 0,
    };
    let gc = document.createElement("script");
    gc.type = "text/javascript";
    gc.async = true;
    gc.src =
      "https://graphcomment.com/js/integration.js?" +
      Math.round(Math.random() * 1e8);
    (
      document.getElementsByTagName("head")[0] ||
      document.getElementsByTagName("body")[0]
    ).appendChild(gc);
  },
};
</script>

<style src="../css/github-markdown.css" />
