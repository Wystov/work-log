<script>
export default {
  data: () => ({
    inputs: {
      done: {
        label: 'Сделал вчера',
        data: [],
      },
      todo: {
        label: 'Сделаю сегодня',
        data: [],
      },
      hard: {
        label: 'Трудности',
        data: [],
      },
    },
  }),
  methods: {
    add(to) {
      const data = { id: Date.now(), text: '' };
      if (to === 'done') data.time = '00:00';
      this.inputs[to].data.push(data);
    },
    remove(from, id) {
      this.inputs[from].data = this.inputs[from].data.filter((item) => item.id !== id);
    },
    copyResult() {
      const text1 = Object.keys(this.inputs).reduce((acc, key) => {
        if (this.inputs[key].data.length) {
          const label = `**${this.inputs[key].label}**\n`;
          // eslint-disable-next-line no-return-assign
          const text = this.inputs[key].data.reduce((a, b) => a += `  • ${b.text} ${b.time ? `🕐${this.getTime(b.time)}` : ''}\n`, '');
          acc += label + text;
        }
        return acc;
      }, '');
      navigator.clipboard.writeText(text1);
    },
    reset() {
      Object.keys(this.inputs).forEach((type) => {
        this.inputs[type].data = [];
      });
    },
    getTime(value) {
      return value.split(':').map((val, i) => (val !== '00' ? `${parseInt(val, 10)}${i === 0 ? 'ч' : 'м'}` : ''))
        .filter(Boolean)
        .join(' ');
    },
  },
  created() {
    const fromLs = localStorage.getItem('work-log: inputs');
    if (fromLs) this.inputs = JSON.parse(fromLs);

    window.addEventListener('beforeunload', () => {
      Object.keys(this.inputs).forEach((type) => {
        this.inputs[type].data = this.inputs[type].data.filter((item) => item.text.length);
      });
      localStorage.setItem('work-log: inputs', JSON.stringify(this.inputs));
    });
  },
};
</script>

<template>
  <div class="container">
    <form class="form">
      <fieldset v-for="item, key in inputs" :key="key">
        <legend>
          {{ item.label }}
        </legend>
        <template v-for="field in item.data" :key="field.id">
          <button type="button" @click="remove(`${key}`, field.id)">-</button>
          <label>
            <textarea v-model="field.text" />
            <input type="time" v-if="field.time !== undefined" v-model="field.time">
          </label>
        </template>
        <button type="button" @click="add(`${key}`)">+</button>
      </fieldset>
    </form>
    <div class="result">
      <h1>worklog</h1>
      <div ref="copy">
        <div v-if="inputs.done.data.length">
          <h3><b>Сделал вчера</b></h3>
          <p v-for="item in inputs.done.data" :key="item.id">
            &nbsp;&nbsp;&nbsp;&nbsp;• {{ item.text }}
            <span v-if="item.time !== '00:00'">🕐{{ getTime(item.time) }}</span>
          </p>
        </div>
        <div v-if="inputs.todo.data.length">
          <h3>Сделаю сегодня</h3>
          <p v-for="item in inputs.todo.data" :key="item.id">
            &nbsp;&nbsp;&nbsp;&nbsp;• {{ item.text }}
          </p>
        </div>
        <div v-if="inputs.hard.data.length">
          <h3>Трудности</h3>
          <p v-for="item in inputs.hard.data" :key="item.id">
            &nbsp;&nbsp;&nbsp;&nbsp;• {{ item.text }}
          </p>
        </div>
      </div>
      <button type="button" @click="copyResult">Copy</button>
      <button type="button" @click="reset">Reset</button>
    </div>
  </div>
</template>

<style>
body {
  margin: 0;
  font-size: 20px;
}

.container {
  padding-top: 10vh;
  width: 98vw;
  max-width: 1200px;
  height: 90vh;
  margin: auto;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 40px;
}

.form, .result {
  width: 100%;
}

fieldset {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 20px;
}

label {
  border-bottom: 1px solid black;
}

label > * {
  margin-bottom: 20px;
}

button {
  align-self: flex-start;
}

textarea {
  width: 100%;
  resize: none;
}

h1 {
  margin: 0;
  text-decoration: underline;
}

</style>
