<template>
  <ideas-list-page
    :initial-ideas="ideas"
    :initial-next-token="nextToken"
    title="SAVED IDEAS"
    :end-point="endPoint"
    :end-point-name="endPointName"
    :allow-order="false"
    show-author
  ></ideas-list-page>
</template>

<script>
import ideasListPage from '@/components/ideasList/ideasListPage'
import loadIdeas from '@/components/ideasList/loadIdeas'
import getSavedIdeas from '~/graphql/query/getSavedIdeas'

export default {
  components: { ideasListPage },
  middleware: 'authenticated',
  asyncData({ app, store }) {
    return loadIdeas(app.$amplifyApi, 'getSavedIdeas', getSavedIdeas)
  },
  data() {
    return {
      endPoint: getSavedIdeas,
      endPointName: 'getSavedIdeas'
    }
  }
}
</script>
