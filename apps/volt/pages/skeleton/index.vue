<template>
    <DocComponent title="Vue Skeleton Component" header="Skeleton" description="Skeleton is a placeholder to display instead of the actual content." :componentDocs="docs" />
</template>

<script setup lang="ts">
import CardDoc from '@/doc/skeleton/CardDoc.vue';
import DownloadDoc from '@/doc/skeleton/DownloadDoc.vue';
import ImportDoc from '@/doc/skeleton/ImportDoc.vue';
import ListDoc from '@/doc/skeleton/ListDoc.vue';
import ShapesDoc from '@/doc/skeleton/ShapesDoc.vue';
import { ref } from 'vue';

const docs = ref([
    {
        id: 'dowload',
        label: 'Download',
        component: DownloadDoc
    },
    {
        id: 'import',
        label: 'Import',
        component: ImportDoc
    },
    {
        id: 'shapes',
        label: 'Shapes',
        component: ShapesDoc
    },
    {
        id: 'card',
        label: 'Card',
        component: CardDoc
    },
    {
        id: 'list',
        label: 'List',
        component: ListDoc
    }
]);
</script>
