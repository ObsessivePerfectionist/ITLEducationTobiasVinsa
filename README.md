# ITLEducationTobiasVinsa

# Different themes for bookshelves:
I would most likely have some tag on each book item in the tables that has a winter or hallowen tag etc. Could be named theme. Using this I would just load the items the same way as the paginator below is described. Could be enums so we can map it to a specific Id i.e winter has the enum value on 1 etc. I was only planning that the single bookshelf should change but maybe there should be an entire page i.e winter section or summer section. If it is the bookshelf only I would assume the rendering of the bookshelf would need to be different and I don’t know how hard that is to implement frontend side with the different containers in the same bigger container. So we load in all books that we should show i.e a chunk with the specific theme for the container and then move to the next container.

# Pagination:
For the pagination part I would use a paginator. This will allow us to select a page and then some results. For better UX we should also have an adaptive viewer that allows the user to select how many items they would like to view. 10/25/50 etc. Now that I looked closer it said infinite scrolling which means we remove the item selector and it should just scroll infinitly. Instead now we should have the backend load items in chunks. That is load 30 items for example and have a cursor that knows what we have loaded in. “LastEvaluatedKey” is a common term for this when using dynamoDb. So the next one will load in next 30 items instead of then repeating the same 30 items again.

Then we need some way of detecting when a user is at the bottom of the first loaded in items. This can be done in some ways but you can use a detection on where the user have their cursor. If it is at the bottom of the screen we can send the next request. Preferably on the container actually because otherwise it might load when the user is just at the bottom of the screen and hovering there. Now that I think about it and look in the example this view seems to have a next button to the right which would trigger an additional load. So instead we seem to load new bookshelves when scrolling down but the button next loads in new books in the same theme. So I would assume we load in more bookshelves books with themes below and books in the same theme when clicking on the right button. So we would need to think about these two cursors then.

# Language support:
For the books part I would probably choose to load all the books in one language maybe English since it is universal but could be adapted depending on where the user is accessing from. In spain for example we could choose Spanish as the main loading queue. I.e each book should have some language tag on them in the DB and then we select every Spanish one. This could have some drawbacks since I would assume most book adaptations exists in English. Then we might now show all the books in the beginning. But moving on when we show all the books we should then show, once the user clicks on a book the languagues this book is in should be available. So how do we know which books we should handle here. We could use some other table containing each book that is from a different language. However this would most likely result in several tables since we might have over 1000+ books. Instead giving each book in the book table an ID or the English name of the book for each book that is the same. 

I.e if we have snow white and snövit each of these additions in the table should have a shared column. Or even simpler we could add a list to the table with each language this book is available in and then simply display that it exists in this language. The other option would allow us to load in the books directly but this option would only show the user that it is available.
It might however be best to skip the tables part and instead have some unique identifier for a book group. I.e all books that belong to snow white should have some unique identifier. So if we have an entry för snövit and one entry for snow white with different id’s the should share a book group id. That will allow us to know what to query the db for more and could maybe show it in some container beside the book someone selected. 

So for the prioritization part. What seems most important for me is the scrolling for a good UX or the language indicator since some themes are towards kids and displaying for some parent or teacher that the book exists in their language is also good value. The themes are not that important in my opinion compared to the others.
So priority

1. Paginator
2. Language indicator
3. Themes

# Wireframe:
https://wireframe.cc/hEl3yq
---------------------------------------------------------
|  Header                                                 |
 ---------------------------------------------------------
|  [ Search bar ]  [ Filter: Theme ]  [ Filter: Language ]|
 ---------------------------------------------------------
|  [Book A]  [Book B]  [Book C]  [Book D]                 |
|  [Book E]  [Book F]  [Book G]  [Book H]                 |
|  [Book I]  [Book J]  [Book K]  [Book L]                 |
 ---------------------------------------------------------
|                 (loading next results...)              |
 ---------------------------------------------------------
|  [Book M]  [Book N]  [Book O]  [Book P]                 |
 ---------------------------------------------------------
|                 You've reached the end.                |
 ---------------------------------------------------------

# Books table should have:
BookId (PK)  
BookGroupId  
Title  
Language  
Theme                 -- optional, e.g. “winter”, “summer”  
CreatedAt             -- used for sorting/pagination  

# Technical decisions
Technical decisions where described above in the free text.

# Release plan:
Start with ensuring everyone on the team knows what to do. After that start working on both frontend and backend at the same time. As long as data is clearly stated what is expected it should work well.

We will need an endpoint for the books get/books?/limit=30(or something else)&cursor=at&theme  
Best to use dynamoDb built in cursor. So I think we can use the same endpoint to load the different items just that we need to keep the cursor separate and the theme should the same one if we click next button or for the next container below a new theme.

Add some logging so that we see if any unexpected errors occur with reasons and request if possible.

Then frontend should be built up like it looks in the screenshots with the addition that if someone clicks on the next button or the below screen one different cursors should be used to send a request to the backend.

Then we should test it and see that the behaviour works. Setting up backend tests to test small business logic things. And for frontend check that it looks good.



# Psuedo Code:

<template>
  <div class="bookshelf">
    <h2>{{ themeTitle }}</h2>
    <div class="books-grid">
      <div v-for="book in books" :key="book.id" class="book-card">
        {{ book.title }}
      </div>
    </div>
    <button v-if="!finished && !loading" @click="loadBooks">Next</button>
    <div v-if="loading">Loading...</div>
    <div v-if="finished">You've reached the end.</div>
  </div>
</template>

<script>
export default {
  props: ['themeTitle', 'themeKey'],
  data() {
    return {
      books: [],
      cursor: null,
      loading: false,
      finished: false
    }
  },
  mounted() {
    this.loadBooks()
  },
  methods: {
    async loadBooks() {
      if (this.loading || this.finished) return
      this.loading = true

      const response = await fetch(`/books?theme=${this.themeKey}&limit=30&cursor=${this.cursor || ''}`)
      const data = await response.json()

      this.books.push(...data.items)

      this.cursor = data.nextCursor
      if (!data.nextCursor) this.finished = true

      this.loading = false
    }
  }
}
</script>


Endpoint: GET /books
Query parameters:
- theme: string
- limit: number
- cursor: string (optional)

Response:
{
  items: [
    { id, title, theme, createdAt, bookGroupId }
  ],
  nextCursor: string | null
}

