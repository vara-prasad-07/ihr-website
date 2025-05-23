<script setup>
import { useRoute, useRouter } from 'vue-router'
import { ref, inject, watch, onMounted } from 'vue'
import IypController from '@/components/controllers/IypController.vue'
import IypGenericTreemapChart from '@/components/charts/IypGenericTreemapChart.vue'
import IypGenericBarChart from '@/components/charts/IypGenericBarChart.vue'

const iyp_api = inject('iyp_api')

const props = defineProps(['pageTitle', 'hostName'])

const route = useRoute()
const router = useRouter()

const rankings = ref({
  data: [],
  show: false,
  loading: true,
  query: `MATCH (:HostName {name: $hostname})-[:PART_OF]-(:DomainName)-[rr:RANK]-(r:Ranking)
    RETURN DISTINCT r.name AS rank_name, rr.rank AS rank, 1/(1+toFloat(rr.rank)) AS inv_rank`,
  columns: [
    {
      name: 'Ranking Name',
      label: 'Ranking Name',
      align: 'left',
      field: (row) => row.rank_name,
      format: (val) => `${val}`,
      sortable: true,
      description:
        'Name of the ranking. Different rankings have different meanings, please see the page corresponding to each ranking for more details.'
    },
    {
      name: 'Rank',
      label: 'Rank',
      align: 'left',
      field: (row) => Number(row.rank),
      format: (val) => `${val}`,
      sortable: true,
      description: 'Position in the ranking.'
    }
  ],
  pagination: {
    sortBy: 'Rank', //string column name
    ascending: true //boolean
  }
})

const load = () => {
  rankings.value.loading = true
  // Run the cypher query
  let query_params = { hostname: props.hostName }
  iyp_api.run([{ statement: rankings.value.query, parameters: query_params }]).then((results) => {
    rankings.value.data = results[0]
    rankings.value.loading = false
  })
}

watch(
  () => props.hostName,
  () => {
    load()
  }
)

onMounted(() => {
  load()
})
</script>

<template>
  <IypController
    :data="rankings.data"
    :columns="rankings.columns"
    :loading-status="rankings.loading"
    :cypher-query="rankings.query.replace(/\$(.*?)}/, `'${hostName}'}`)"
    :pagination="rankings.pagination"
  />
</template>
