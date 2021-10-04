<script>
import kegiatanJson from '@/assets/kegiatan.json'

export default {
  data: () => ({
    dataKegiatan: Object
  }),
  methods: {
    loadData () {
      console.log(kegiatanJson)
    }
  }
}
</script>
