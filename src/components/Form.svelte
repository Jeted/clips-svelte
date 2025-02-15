<script lang="ts">
  import moment from 'moment';
  import Input from './Form/Input.svelte';
  import Button from './Form/Button.svelte';
  import { get } from 'svelte/store';
  import { Form } from '../misc/types';
  import { sortClips } from 'src/helpers';
  import { getClips, getUser } from '../requests';
  import { loading, params, clips } from '../misc/store';

  enum DATE {
    FORMAT = 'DD/MM/YYYY',
  }

  let form: Form = {
    from_date: '26/05/2016',
    to_date: moment(new Date()).format(DATE.FORMAT),
    channel: '',
  };

  async function handleSubmit(e: any) {
    clips.set([]);

    form = {
      from_date: e.target[0].value,
      to_date: e.target[1].value,
      channel: e.target[2].value,
    };

    if (form.channel === '') return;
    loading.set(true);

    getUser(form.channel, true)
      .then((user) => {
        params.set({
          broadcaster_id: user.userId,
          first: 100,
          started_at: moment(form.from_date, DATE.FORMAT).toISOString(),
          ended_at: moment(form.to_date, DATE.FORMAT).add(1, 'day').toISOString(),
        });
      })
      .catch(() => loading.set(false));
  }

  params.subscribe((value) => {
    if (!value) return;
    loading.set(true);

    const url = new URL('https://api.twitch.tv/helix/clips');
    Object.entries(get(params)).forEach(([param, value]) => {
      url.searchParams.set(param, value.toString());
    });

    getClips(url).then(([data, pagination]) => {
      loading.set(false);
      clips.update((values) => sortClips(values.concat(data)));
      if (pagination) {
        params.update((values) => ({ ...values, after: pagination }));
      }
    });
  });
</script>

<section>
  <form on:submit|preventDefault={handleSubmit}>
    <Input placeholder="From..." value={form.from_date} />
    <Input placeholder="To..." value={form.to_date} />
    <Input placeholder="Channel" value={form.channel} />
    <Button />
  </form>
</section>

<style lang="scss">
  @import '../styles/colors.scss';

  section {
    background-color: $secondary-color;
    border-radius: 10px;
    height: 35px;
    margin-bottom: 10px;
    padding: 8px;

    form {
      display: inline-flex;
      height: 35px;
      position: relative;
      width: 100%;

      :global(input),
      :global(button) {
        appearance: none;
        background: $base-color;
        border: none;
        border-radius: 4px;
        color: $text-color;
        font-family: Inter;
        font-size: 14px;
        padding: 5px 15px;
        width: 28%;
        &::placeholder {
          color: $text-color;
        }
      }

      :global(input) {
        &:disabled {
          cursor: not-allowed;
        }
        &:focus {
          box-shadow: 0 0 0 3px $transparent-white;
          outline: none !important;
        }
      }
    }
  }
</style>
