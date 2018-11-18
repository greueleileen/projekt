<template>
    <div>
        <h2>Artikel</h2>
        <form @submit.prevent="addArticle" class="mb-3">
            <h4>Füge einen neuen Artikel hinzu.</h4>
            <div class="form-group">
                <input type="text" class="form-control" placeholder="Titel" v-model="article.title">
            </div>
            <div class="form-group">
                <textarea class="form-control" placeholder="Artikelbeschreibung" v-model="article.body"></textarea>
            </div>
            <button type="submit" class="btn btn-light btn-block">Speichern</button>
        </form>
        <div class="card card-body mb-2" v-for="article in articles" v-bind:key="article.id">
            <h3>{{ article.title}}</h3>
            <p>{{ article.body}}</p>
            <hr>
            <button class="btn btn-danger mb-2" @click="deleteArticle(article.id)">Artikel löschen</button>
            <button class="btn btn-primary" @click="editArticle(article)">Artikel bearbeiten</button>
        </div>
        <nav aria-label="...">
            <ul class="pagination">
                <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item">
                    <a class="page-link" href="#" tabindex="-1" @click="fetchArticles(pagination.prev_page_url)"><</a>
                </li>
                <li class="page-item disabled"><a class="page-link text-dark" href="#">Seite {{ pagination.current_page }} von {{ pagination.last_page }}</a></li>

                <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">></a>
                </li>
            </ul>
        </nav>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                articles: [],
                article: {
                    id: '',
                    title: '',
                    body: ''
                },
                article_id: '',
                pagination: {},
                edit: false
            }
        },

        created(){
            this.fetchArticles();
        },

        methods: {
            fetchArticles(page_url){
                let vm = this;
                page_url =page_url || '/api/articles'
                fetch(page_url)
                    .then(res => res.json())
                    .then(res => {
                        this.articles = res.data;
                        vm.makePagination(res.meta, res.links)
                    })
                    .catch(err => console.log(err));
            },
            makePagination(meta, links){
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    next_page_url: links.next,
                    prev_page_url: links.prev
                }
                this.pagination = pagination;
            },
            deleteArticle(id) {
                if(confirm('Bist du dir sicher, das du diesen Artikel löschen möchtest?')){
                     fetch(`api/article/${id}` ,{
                         method: 'delete'
                    })
                    .then(res => res.json())
                         .then(data => {
                             alert('Artikel gelöscht');
                             this.fetchArticles();
                         })
                         .catch(err => console.log(err));
                }
            },
            addArticle() {
                if(this.edit === false) {
                    //Add new article
                    fetch('api/article', {
                        method: 'post',
                        body: JSON.stringify(this.article),
                        headers: {
                            'content-type': 'application/json'
                        }
                    })
                        .then(res => res.json())
                        .then(data => {
                            this.article.title = '';
                            this.article.body = '';
                            alert('Artikel hinzugefügt');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                } else {
                    //Update article
                    fetch('api/article', {
                        method: 'put',
                        body: JSON.stringify(this.article),
                        headers: {
                            'content-type': 'application/json'
                        }
                    })
                        .then(res => res.json())
                        .then(data => {
                            this.article.title = '';
                            this.article.body = '';
                            alert('Artikel geändert');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                }
            },
            editArticle(article) {
                this.edit = true;
                this.article.id = article.id;
                this.article.article_id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            }
        }
    };
</script>

<style scoped>

</style>