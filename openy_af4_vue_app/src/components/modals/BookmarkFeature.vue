<template>
  <Modal
    id="activity-finder-bookmark-feature"
    :title="'New feature!' | t"
    :value="visibility"
    narrow
  >
    <div class="bookmark-feature-modal-content">
      <BookmarkFeatureDescription />
    </div>
  </Modal>
</template>

<script>
import Modal from '@/components/modals/Modal.vue'
import BookmarkFeatureDescription from '@/components/modals/BookmarkFeatureDescription.vue'

export default {
  name: 'BookmarkFeature',
  components: {
    Modal,
    BookmarkFeatureDescription
  },
  data() {
    let data = {
      cookieName: 'af_bookmark_feature_modal_shown'
    }
    data.visibility = this.getCookie(data.cookieName) !== '1'

    return data
  },
  mounted() {
    const maxAge = 365 * 24 * 60 * 60
    document.cookie = encodeURIComponent(this.cookieName) + '=1; max-age=' + maxAge
  }
}
</script>

<style lang="scss">
.bookmark-feature-modal-content {
  padding-top: 20px;
}
</style>
