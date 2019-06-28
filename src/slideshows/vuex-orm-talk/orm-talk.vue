<template lang="pug">
#orm-slideshow.eg-theme-refuel-dark
  .eg-slideshow
    slide(leave='fadeOut')
      h2
        img.logo(src='./assets/logo-vuex-orm.png')
        | Vuex ORM
      p.center Reducing Relational Complexity
      p.center.repo https://github.com/vuex-orm/vuex-orm
    slide(enter='fadeIn' leave='fadeOut')
      h3 Why?
      p.center Vuex ORM is an attempt to deal with heavily nested data structures
        |  that are often returned by APIs without having to write huge
        |  amounts of boilerplate, reducers, and combination logic throughout your app.
    slide(enter='fadeIn' leave='fadeOut')
      h3 Highlights:
      .center
        p Write fewer actions and mutations
        p Dramatic Reduction in Component code
        p Automatic data normalization
        p Relationships are fully supported
        p Plugin system (axios, graphql)
        p Full Typescript Support
    slide(enter='fadeIn' leave='fadeOut')
      h3 Overview of Structure
      ul.list
        li
          b Database
          p Registers all of the models with the module
        li
          b Models
          p Similar to other ORM models
        li
          b Vuex Store - entities
          p Model data is stored into a single module in Vuex
    slide(enter='fadeIn' leave='fadeOut')
      h3 Basic Structure - Database
      .code-block
        .column
          p Models are registered to VuexORM via a database instance
          p State is stored in the entities vuex module namespace by default
        .column
          eg-code-block(lang='js').
            const database = new VuexORM.Database()
            database.register(User)
            database.register(Post)
          eg-code-block(lang='js').
            state {
              entities: {
                users: {...},
                posts: {...},
                ...
              }
            }
    slide(enter='fadeIn' leave='fadeOut')
      h3 Basic Structure - Models
      .code-block
        ul.column
          li
            b Entity
            |  defines the vuex state node for storage
            eg-code-block(lang='js').
              store.state.entities.users
          li
            b Fields
            |  are the data schema for the component
        eg-code-block.column(lang='js').
          import { Model } from '@vuex-orm/core'

          class User extends Model {
            static entity = 'users'

            static fields () {
              return {
              id: this.attr(null),
              name: this.attr('')
            }
          }

    slide(enter='fadeIn' leave='fadeOut')
      h3 Basic Structure - Models
      p.center Relationships
      .code-block
        .column
          p Supports the following relations:
          ul
            li One to One
            li One to Many
            li One to One Inverse
            li Has Many By
            li Has Many Through
            li Polymorphic Relations
        eg-code-block.column(lang='js').
          class User extends Model {
            static entity = 'users'

            static fields () {
              return {
              id: this.attr(null),
              name: this.attr(''),
              profile_id: this.attr(null),
              profile: this.hasOne(Profile, 'id', 'profile_id')
              posts: this.hasMany(Posts, 'user_id', 'id')
            }
          }

          User.query().with('posts').get()

    slide(enter='fadeIn' leave='fadeOut')
      h3 Basic Structure - Models
      p.center Interacting with Data
      .code-block
        eg-code-block.column(lang='js').
          // Destructive
          User.create({ data: this.data })

          // Insert
          User.insert({ data: this.data })

          // Insert /w Update
          User.insertOrUpdate({ data: this.data })

          // Delete
          User.delete(1)
          User.delete((user => user.name === 'Wilson'))
          user.deleteAll()

          // Query Builder
          User.find(1)
          User.findIn([1, 2, 3])
          User.query().where('name', 'Wilson').with('posts').get()

          User.query().where(user => user.permissions > ADMIN).get()
    slide(enter='fadeIn' leave='fadeOut')
      h3 Reduction in Vuex Boilerplate
      p.center Vuex Actions can get a bit out of hand
      eg-code-block(lang='js').
          async fetchItem(
            { commit, dispatch, rootState },
            payload: IApiPayload
          ): Promise {
            const item = await api(rootState.auth).get(`/api/items/${payload.id}`, {
                params: {
                expand: 'tags, posts',
              },
            });

            // this is an external action called from user module
            const usersData = await dispatch(
            'user/fetchUsers',
              { fields: 'id, email' },
              { root: true }
            );

            item.data.posts = attachUsersToPosts(item.data.posts, usersData.data);
            item.data.posts = attachUsersToPosts(
              item.data.posts,
              usersData.data,
              'manager',
              'userProfile'
            );

            commit(items.mutations.setItem, item.data);
         }
    slide(enter='fadeIn' leave='fadeOut')
      h3 Reduction in Vuex Boilerplate
      p.center Mutations are often just noise
      eg-code-block.center-code-block(lang='js').
        [items.mutations.setItem](state, item: IItem) {
          state.items.push(item);
        },
      p.center Or Expensive
      eg-code-block.center-code-block(lang='js').
          [items.mutations.updateItems](state, mergingItems: object[]) {
            // Merges items by comparing ids
            state.items = mergeTo(R.prop('id'), state.items, mergingItems);
          },
    slide(enter='fadeIn' leave='fadeOut')
      h3 Reduction in Vuex Boilerplate
      p.center Vuex ORM reduces this entire process to:
      eg-code-block.center-code-block(lang='js').
        async fetchItems(
          { commit, dispatch, rootState },
            payload: IApiPayload
        ): Promise {
          const item = await api(rootState.auth).get(`/api/items/${payload.id}`, {
            params: {
              expand: 'tags, posts',
            },
          });

          // this is an external action called from user module
          await dispatch(
          'user/fetchUsers',
            { fields: 'id, email' },
            { root: true }
          );

          await Item.insertOrUpdate({ data: item })
        }
      p.center No mutations are necessary thanks to relations. They automatically populate.
    slide(enter='fadeIn' leave='fadeOut')
      h3 Readability in Component Code
      p.center Maybe you have a page with some filters
      eg-code-block.center-code-block(lang='js').
        computed: {
          ...mapState('items', ['items'])
          blueVisibleItems() {
            return items.filter((item) => {
              return item.color === 'blue' && item.visible
            })
          },
        }
      p.center Becomes:
      eg-code-block.center-code-block(lang='js').
        computed: {
          blueVisibleItems() {
            return Item.query()
              .where('color', 'blue')
              .where('visible', true)
              .get()
          }
        }
    slide(enter='fadeIn' leave='fadeOut')
      h3 Component Code Reduction
      p.center Construct Objects via Relations
      eg-code-block.center-code-block(lang='js').
        computed: {
          question() {
            return Question.query()
              .with('assignments, scoring, user')
              .find(this.selectedQuestion)
          }
        }
      p.center Build detailed filters
      eg-code-block.center-code-block(lang='js').
        selectedQuestions() {
          const query = Question.query()
            .withAllRecursive()

          if (this.filters.assigned) {
            query.where('assigned', true)
          }

          if (this.filters.withAttachments) {
            query.where(question => question.document_id !== null)
          }

          return query.get()
        }
    slide(enter='fadeIn' leave='fadeOut')
      h3 Plugins
      p.center Robust plugin architecture lets you hook into methods and lifecycle
      p.center Axios Plugin allows you to specify api calls from within the component
      p.center GraphQl Plugin writes your queries & mutations for you based on the model
    slide(enter='fadeIn' leave='fadeOut')
      h3 Demo
</template>

<script>
import eagle from 'eagle.js'

export default {
  mixins: [eagle.slideshow],
  infos: {
    title: 'Vuex ORM',
    description: 'Vuex ORM talk',
    path: 'vuex-orm'
  }
}
</script>

<style lang="scss">
@import 'node_modules/eagle.js/dist/themes/refuel-dark/refuel-dark';
</style>

<style lang="scss" scoped>
.quarter {
  width: 25%;
  text-align: center;
  p {
    margin-top: 0;
    text-align: center;
  }
  h4 {
    margin-top: 0;
    margin-bottom: 0;
  }
}

.half {
  width: 50%;
  float: right;
}

.logo {
  padding-top: 4rem;
  width: 8rem;
  height: 8rem;
}

.repo {
  font-size: 1.5rem;
}

.list {
  flex: 50%;
  padding-left: 20rem;
}

.center-code-block {
  flex: 50%;
  padding-left: 25rem;
}

.code-block {
  padding-left: 8rem;
  padding-right: 8rem;
  display: flex;

  .column {
    flex: 50%;
  }
}
</style>
