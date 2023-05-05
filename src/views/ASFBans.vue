<template>
  <main class="main-container main-container--fullheight bots-overview">
    <!-- <div class="container"> -->
      <template v-if="loading">
        <h3 class="subtitle">
          <FontAwesomeIcon icon="spinner" size="lg" spin></FontAwesomeIcon>
        </h3>
      </template>
      <template v-else-if="!bannedIps.length">
        <div class="container">
          <div class="asf-bans">
            <span>{{ $t('asf-bans-empty') }}</span>
            <div class="form-item">
              <button class="button button--confirm" @click="refresh()">
                <span>{{ $t('asf-bans-refresh') }}</span>
              </button>
            </div>
          </div>
        </div>
      </template>
      <template v-else>
        <div class="form-item">
          <div class="form-item__buttons">
            <button class="button button--confirm pull-right" @click="refresh()">
              <span>{{ $t('asf-bans-refresh') }}</span>
            </button>
            <button class="button button--cancel" @click="removeBan()">
              <FontAwesomeIcon v-if="removingAll" icon="spinner" spin></FontAwesomeIcon>
              <span v-else>{{ $t('asf-bans-remove-all') }}</span>
            </button>
          </div>
        </div>

        <div class="bots">
          <!-- <div v-for="bannedIp in bannedIps" :key="bannedIp" class="ban">
            <p>IP: {{ bannedIp }}</p>
            <button class="button button--confirm" @click="removeBan(bannedIp)">
              <FontAwesomeIcon v-if="removing" icon="spinner" spin></FontAwesomeIcon>
              <span v-else>{{ $t('asf-bans-remove') }}</span>
            </button>
          </div> -->
          <div v-for="bannedIp in bannedIps" :key="bannedIp" class="bot-placeholder status--disabled">
            <!-- <div class="bot-placeholder__button bot-placeholder__button--add">
              <span>Ip: {{ bannedIp }}</span>
            </div> -->

              <!-- <FontAwesomeIcon icon="plus" class="bot-placeholder__icon"></FontAwesomeIcon>
              <span class="bot-placeholder__name">{{ $t('bot-new') }}</span> -->

              <!-- <FontAwesomeIcon v-if="removing" icon="spinner" spin></FontAwesomeIcon> -->
              <!-- <span v-else>{{ $t('asf-bans-remove') }}</span> -->
              
              

            <div class="bot__status">
              <span class="bot__status-property bot__status-property--name">Ip: {{ bannedIp }}</span>
            </div>

            <div class="bot__actions pull-right">
              <span v-tooltip="$t('asf-bans-remove')" class="bot__action" @click="removeBan(bannedIp)"><FontAwesomeIcon icon="trash"></FontAwesomeIcon></span>
            </div>
          </div>
        </div>
      </template>
    <!-- </div> -->
  </main>
</template>

<script>
  export default {
    name: 'ASFBans',
    metaInfo() {
      return {
        title: this.$t('asf-bans'),
      };
    },
    data() {
      return {
        loading: false,
        removing: false,
        removingAll: false,
        bannedIps: [],
      };
    },
    created() {
      this.refresh();
    },
    methods: {
      async removeBan(bannedIp = null) {
        if (this.removing || this.removingAll) return;

        if (bannedIp) this.removing = true;
        else this.removingAll = true;

        try {
          const endpoint = (bannedIp) ? `ipc/bans/${bannedIp}` : 'ipc/bans';
          await this.$http.del(endpoint);

          if (bannedIp) this.$success(this.$t('asf-bans-deleted', { ip: bannedIp }));
          else this.$success(this.$t('asf-bans-deleted-all'));

          this.refresh();
        } catch (err) {
          this.$error(err.message);
        } finally {
          this.removing = false;
        }
      },
      async refresh() {
        if (this.loading) return;
        this.loading = true;

        try {
          this.bannedIps = await this.$http.get('ipc/bans');
        } catch (err) {
          this.$error(err.message);
        } finally {
          this.loading = false;
        }
      },
    },
  };
</script>

<style lang="scss">
  .asf-bans {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 1em;

    button {
      margin-top: 1em;
    }

    .ban {
      display: flex;
      align-items: center;

      button {
        margin: 1em;
      }
    }
  }


  .bots-overview {
    display: grid;
    grid-gap: 1rem;
    grid-template-columns: 1fr;
    grid-template-rows: min-content min-content;
  }

  .bots {
    display: grid;
    grid-gap: 1em;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    min-height: 0;
    margin-bottom: 1em;

    @media screen and (max-width: 1366px) {
      grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    }

    @media screen and (max-width: 300px) {
      grid-template-columns: 1fr;
    }
  }

  .bot-placeholder {
    align-items: center;
    background: var(--color-background-light);
    border-radius: 0 0 4px 4px;
    border-top: 3px solid var(--color-status);
    cursor: pointer;
    display: flex;
    height: 2.25em;
    justify-content: center;
    padding: 0.5em;
    transition: border .3s;

    &--big {
      font-size: 1.25rem;
      height: auto;
    }
  }

  .bot-placeholder__button {
    align-items: center;
    display: flex;
  }

  .bot-placeholder__name {
    font-size: 0.8em;
    font-weight: 600;
  }

  .bot-placeholder__icon {
    margin-right: 0.5em;
  }

  .bot {
    background: var(--color-background-light);
    border-radius: 0 0 4px 4px;
    border-top: 3px solid var(--color-status);
    display: grid;
    grid-template-areas: 'avatar meta buttons';
    grid-template-columns: min-content 1fr auto;
    padding: 0.5em;
    transition: border .3s;
  }

  .bot__avatar {
    cursor: pointer;
    display: block;
    height: 2.25em; // (1em + 0.8em) * 1.25
    padding-right: 0.5em;
    width: 2.25em;
  }

  .bot__status {
    cursor: pointer;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    margin: 0.5em;
  }

  .bot__status-property {
    display: inline-block;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .bot__status-property--name {
    font-weight: 600;
  }

  .bot__status-property--text {
    font-size: 0.8em;
    font-style: italic;
  }

  .bot__actions {
    align-items: center;
    display: flex;
    grid-area: buttons;
    justify-content: space-around;
  }

  .bot__action {
    color: var(--color-text-disabled);
    cursor: pointer;
    display: block;
    padding: 0.5em;
    transition: color .3s;

    &:hover {
      color: var(--color-text-dark);

      .app--dark-mode & {
        color: var(--color-text);
      }
    }
  }
</style>
