<script setup lang="ts">
import grapesjs, { Block, BlockProperties, Component, Editor } from 'grapesjs';
import 'grapesjs/dist/css/grapes.min.css';
import { onMounted, reactive } from "vue";

const titleItemScript = function({ userName }: { userName: string }) {
  console.log(`Hello: ${userName}`);
};

const dbgCmpnnt = (component: Component) => console.log(component.toHTML(), component.toJSON());
const blocks: BlockProperties[] =  [{
  id: 'input',
  label: 'input',
  content: '<input name="input_name" placeholder="default" />'
}];

const adadMenuSectionComponent = (editor: Editor) => {
  editor.DomComponents.addType('custom_component', {
    model: {
      defaults: {
        testprop: 1,
      },
      init() {
        console.log('Local hook: model.init', this);
        this.listenTo(this, 'change:testprop', this.handlePropChange);
      },
      // updated(property, value, prevValue) {
      //   console.log('Local hook: model.updated',
      //     'property', property, 'value', value, 'prevValue', prevValue);
      // },
      // removed() {
      //   console.log('Local hook: model.removed');
      // },
      // handlePropChange() {
      //   console.log('The value of testprop', this.get('testprop'));
      // }
    },
    view: {
      init(opts) {
        const numberOfMenuItemsToCreate = (document.getElementById("number_of_menu_items") as HTMLInputElement).value;
        console.log('Local hook: view.init');
        opts.model.components().set([...Array(+numberOfMenuItemsToCreate)].map((_, i) => ({
          tagName: 'div',
          content: `<h${i + 1}>Menu Item ${i + 1}</h${i + 1}>`,
        })));
      },
      onRender() {
        console.log('Local hook: view.onRender');
      },
    },
  });

  // A block for the custom component
  editor.BlockManager.add('custom_component', {
    id: 'grouped_item',
    label: 'Section With Menu Items',
    content: {
      type: "custom_component",
      name: "grouped_component",
      tagName: 'div',
      draggable: true,
      attributes: { 'proxy-attribute': 'proxy-value' },
      components: [
        {
          tagName: 'div',
          content: '<h1>Menu Item 1 ID: </h1>',
        }
      ],
    }
  });
};

const addTraits = (editor: Editor) => {
  editor.Components.addType('input', {
      isComponent: el =>   {
        return el.tagName === 'INPUT'
      },
    model: {
      init() {
        // Also the listener changes from `change:attributes:*` to `change:*`
        this.on('change:placeholder', this.handlePlhChange);
      },
      defaults: {
        traits: [
          { type: 'text', name: 'name' },
          { type: 'text', name: 'placeholder', label: 'My trait', placeholder: 'Insert text', },
          {
            type: 'select', // Type of the trait
            name: 'type', // (required) The name of the attribute/property to use on component
            label: 'Type', // The label you will see in Settings
            options: [
              { id: 'text', label: 'Text'},
              { id: 'email', label: 'Email'},
              { id: 'password', label: 'Password'},
              { id: 'number', label: 'Number' },
            ]
          }, {
            type: 'checkbox',
            name: 'readonly',
        }],
        attributes: { type: 'text', required: true },
      },
    },
  });
};

let editor = reactive<Editor>(null as unknown as Editor);
onMounted(() => {
  editor = grapesjs.init({
    container: '#gjs',
    fromElement: true,
    height: '512px',
    width: 'auto',
    storageManager: false,
    blockManager: { blocks },
  });

  addTraits(editor);
  adadMenuSectionComponent(editor);

  // editor.on(`component:create`, model => console.log('Global hook: component:create', model.get('type'), model));
  // editor.on(`component:mount`, model => console.log('Global hook: component:mount', model.get('type'), model));
  // editor.on(`component:update:testprop`, model => console.log('Global hook: component:update:testprop', model.get('type'), model));
  // editor.on(`component:remove`, model => console.log('Global hook: component:remove', model.get('type'), model));
});

const addToCanvas = () => {
  editor.addComponents(`<div>Added directly <i>to</i> canvas</div>`)[0];
};

const addToSelected = () => {
  const selected = editor.getSelected();
  if (selected) {
    editor.getSelected()?.append(`...`);
  }

  console.log(editor.getWrapper()?.components().forEach(dbgCmpnnt));
};

const changeUserNameInCanvas = () => {
  editor.getComponents().forEach((comp: Component) => {
    if (comp.attributes.type === "title_item") {
      const newUserName = (document.getElementById("username") as HTMLInputElement).value;
      comp.setAttributes({ ...comp.attributes, userName: newUserName });
      comp.components([{
        tagName: 'div',
        content: `Hello User: ${newUserName}!!!`
      }]);
    }
  });
};

const changePriceInBlockPanel = () => {
  editor.Blocks.blocks.forEach((el: Block) => {
    if (el.id === "price_item") {
      const newPrice = `${(document.getElementById("price") as HTMLInputElement).value}$`;

      el.attributes.label = `<u>${newPrice}</u>`;
      el.attributes.content = `<div>${newPrice}</div>`;
    }
  });
};

const addNewBlocksToPanel = () => {
  let userName = "Alexander";
  editor.Components.addType('title_item', {
    model: {
      id: 'title_item',
      label: 'Title Item',
      defaults: {
        userName,
        script: titleItemScript,
        'script-props': ['userName'],
        components: [{
          tagName: 'div',
          content: `Hello User: ${userName}!!!`
        }]
      },
      updated(property: string, value: string, prevValue: string) {
        console.log('property', property, 'value', value, 'prevValue', prevValue);
      }
    }
  });

  editor.BlockManager.add('title_item', {
    id: 'title_item',
    label: 'Title Item',
    content: { type: 'title_item', customProp: 'YYY' }
  });

  editor.BlockManager.add('price_item', {
    id: 'price_item',
    label: '5000$',
    content: {
      tagName: 'div',
      content: "5000$"
    }
  });
}

const addNewMenuItemToCanvas = () => {
  const component = editor.getWrapper()?.components().find(comp => comp.attributes.name === "grouped_component");
  component?.append('<div><h4>New Menu Item</h4><div>')
}
</script>

<template>
  <div id="gjs">
  </div>

  <button @click="addToCanvas">addToCanvas</button>
  <button @click="addToSelected">addToSelected</button>
  <button @click="addNewBlocksToPanel">addNewBlocksToPanel</button>
  <button @click="addNewMenuItemToCanvas">addNewMenuItemToCanvas</button>

  <br />
  <hr />
  <br />

  <button @click="changeUserNameInCanvas">changeUserNameInCanvas</button>
  <button @click="changePriceInBlockPanel">changePriceInBlockPanel</button>
  <br />
  <input id="username" type="text" placeholder="Enter username" value="Viktor" />
  <input id="price" type="number" min="0" placeholder="Enter price" value="5000" />

  <br />
  <hr />
  <br />

  <label>Number of menu items</label>
  <input id="number_of_menu_items" type="text" placeholder="Enter # of menuitems" min="3" value="3" />
</template>
