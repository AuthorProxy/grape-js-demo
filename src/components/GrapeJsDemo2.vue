<script setup lang="ts">
import grapesjs from 'grapesjs';
import 'grapesjs/dist/css/grapes.min.css';
import { onMounted, ref } from "vue";

let editor = ref();
onMounted(() => {
  editor = grapesjs.init({
    // Indicate where to init the editor. You can also pass an HTMLElement
    container: '#gjs',
    // Get the content for the canvas directly from the element
    // As an alternative we could use: `components: '<h1>Hello World Component!</h1>'`,
    fromElement: true,
    // Size of the editor
    height: '512px',
    width: 'auto',
    // Disable the storage manager for the moment
    storageManager: false,
    // Avoid any default panel
    // panels: { defaults: [] },
    layerManager: {
      appendTo: '.layers-container'
    },
    // We define a default panel as a sidebar to contain layers
    panels: {
      defaults: [{
        id: 'layers',
        el: '.panel__right',
        // Make the panel resizable
        resizable: {
          maxDim: 350,
          minDim: 200,
          tc: 0, // Top handler
          cl: 1, // Left handler
          cr: 0, // Right handler
          bc: 0, // Bottom handler
          // Being a flex child we need to change `flex-basis` property
          // instead of the `width` (default)
          keyWidth: 'flex-basis',
        },
      }, {
        id: 'panel-switcher',
        el: '.panel__switcher',
        buttons: [{
          id: 'show-layers',
          active: true,
          label: 'Layers',
          command: 'show-layers',
          // Once activated disable the possibility to turn it off
          togglable: false,
        }, {
          id: 'show-style',
          active: true,
          label: 'Styles',
          command: 'show-styles',
          togglable: false,
        }, {
          id: 'show-traits',
          active: true,
          label: 'Traits',
          command: 'show-traits',
          togglable: false,
        }],
      }, {
        id: 'panel-devices',
        el: '.panel__devices',
        buttons: [{
          id: 'device-desktop',
          label: 'D',
          command: 'set-device-desktop',
          active: true,
          togglable: false,
        }, {
          id: 'device-mobile',
          label: 'M',
          command: 'set-device-mobile',
          togglable: false,
        }]
      }]
    },
    // storageManager: {
    //   type: 'local', // Type of the storage, available: 'local' | 'remote'
    //   autosave: true, // Store data automatically
    //   autoload: true, // Autoload stored data on init
    //   stepsBeforeSave: 1, // If autosave enabled, indicates how many changes are necessary before store method is triggered
    //   options: {
    //     local: { // Options for the `local` type
    //       key: 'gjsProject', // The key for the local storage
    //     },
    //   }
    // },
    deviceManager: {
      devices: [{
        name: 'Desktop',
        width: '',
      }, {
        name: 'Mobile',
        width: '320px',
        widthMedia: '480px'
      }],
    },
    traitManager: {
      appendTo: '.traits-container',
    },
    blockManager: {
      appendTo: '#blocks',
      blocks: [
        {
          id: 'section', // id is mandatory
          label: '<b>Section</b>', // You can use HTML/SVG inside labels
          attributes: { class:'gjs-block-section' },
          content: `<section>
            <h1>This is a simple title</h1>
            <div>This is just a Lorem text: Lorem ipsum dolor sit amet</div>
          </section>`,
        }, {
          id: 'text',
          label: 'Text',
          content: '<div data-gjs-type="text">Insert your text here</div>',
        }, {
          id: 'image',
          label: 'Image',
          // Select the component once it's dropped
          select: true,
          // You can pass components as a JSON instead of a simple HTML string,
          // in this case we also use a defined component type `image`
          content: { type: 'image' },
          // This triggers `active` event on dropped components and the `image`
          // reacts by opening the AssetManager
          activate: true,
        }
      ]
    },
    // The Selector Manager allows to assign classes and
    // different states (eg. :hover) on components.
    // Generally, it's used in conjunction with Style Manager
    // but it's not mandatory
    selectorManager: {
      appendTo: '.styles-container'
    },
    styleManager: {
      appendTo: '.styles-container',
      sectors: [{
          name: 'Dimension',
          open: false,
          // Use built-in properties
          buildProps: ['width', 'min-height', 'padding'],
          // Use `properties` to define/override single property
          properties: [
            {
              // Type of the input,
              // options: integer | radio | select | color | slider | file | composite | stack
              type: 'integer',
              name: 'The width', // Label for the property
              property: 'width', // CSS property (if buildProps contains it will be extended)
              units: ['px', '%'], // Units, available only for 'integer' types
              defaults: 'auto', // Default value
              min: 0, // Min value, available only for 'integer' types
            }
          ]
        },{
          name: 'Extra',
          open: false,
          buildProps: ['background-color', 'box-shadow', 'custom-prop'],
          properties: [
            {
              id: 'custom-prop',
              name: 'Custom Label',
              property: 'font-size',
              type: 'select',
              defaults: '32px',
              // List of options, available only for 'select' and 'radio'  types
              options: [
                { value: '12px', name: 'Tiny' },
                { value: '18px', name: 'Medium' },
                { value: '32px', name: 'Big' },
              ],
          }
        ]
      }]
    },
  });

  editor.Commands.add('show-traits', {
    getTraitsEl(editor) {
      const row = editor.getContainer().closest('.editor-row');
      return row.querySelector('.traits-container');
    },
    run(editor, sender) {
      this.getTraitsEl(editor).style.display = '';
    },
    stop(editor, sender) {
      this.getTraitsEl(editor).style.display = 'none';
    }
  });

  editor.Commands.add('show-layers', {
    getRowEl(editor) { return editor.getContainer().closest('.editor-row'); },
    getLayersEl(row) { return row.querySelector('.layers-container'); },

    run(editor, sender) {
      const lmEl = this.getLayersEl(this.getRowEl(editor));
      lmEl.style.display = '';
    },
    stop(editor, sender) {
      const lmEl = this.getLayersEl(this.getRowEl(editor));
      lmEl.style.display = 'none';
    },
  });

  editor.Commands.add('show-styles', {
    getRowEl(editor) { return editor.getContainer().closest('.editor-row'); },
    getStyleEl(row) { return row.querySelector('.styles-container') },

    run(editor, sender) {
      const smEl = this.getStyleEl(this.getRowEl(editor));
      smEl.style.display = '';
    },
    stop(editor, sender) {
      const smEl = this.getStyleEl(this.getRowEl(editor));
      smEl.style.display = 'none';
    },
  });

  editor.Commands.add('set-device-desktop', {
    run: editor => editor.setDevice('Desktop')
  });

  editor.Commands.add('set-device-mobile', {
    run: editor => editor.setDevice('Mobile')
  });

  editor.on('change:device', () => console.log('Current device: ', editor.getDevice()));

  editor.BlockManager.add('my-block-id', {
    id: 'MyBlockId',
    label: 'MyBlockLabel',
    content: {
      tagName: 'div',
      draggable: true,
      attributes: { 'some-attribute': 'some-value' },
      components: [
        {
          tagName: 'span',
          content: '<b>Some static content</b>',
        }, {
          tagName: 'div',
          components: '<span>HTML at some point</span>',
        }
      ]
    }
  });

  editor.BlockManager.add('rockbot-block-id', {
    id: 'RockbotDynamicId',
    label: 'RockbotDynamicId',
    content: {
      id: 'text',
      label: 'Text',
      content: '<div id="rockbot-ID">Here is a Rockbot ID: ==ID==</div>',
    }
  });

  editor.Panels.addPanel({
    id: 'panel-top',
    el: '.panel__top',
  });

  editor.Panels.addPanel({
    id: 'basic-actions',
    el: '.panel__basic-actions',
    buttons: [
      {
        id: 'visibility',
        active: true, // active by default
        className: 'btn-toggle-borders',
        label: 'Show Lines!!!',
        command: 'sw-visibility', // Built-in command
      }, {
        id: 'export',
        className: 'btn-open-export',
        label: 'Export Me!!!',
        command: 'export-template',
        context: 'export-template', // For grouping context of buttons from the same panel
      }, {
        id: 'show-json',
        className: 'btn-show-json',
        label: 'to JSON!!!',
        context: 'show-json',
        command(editor) {
          editor.Modal.setTitle('Components JSON')
            .setContent(`<textarea style="width:100%; height: 250px;">${JSON.stringify(editor.getComponents(), null, 2)}</textarea>`)
            .open();
        },
      }
    ],
  });

  editor.on('run:export-template:before', opts => {
    console.log('Before the command run');
  });

  editor.on('run:export-template', () => console.log('After the command run'));
  editor.on('abort:export-template', () => console.log('Command aborted'));

  // The `props` argument will contain only the properties you have declared in `script-props`
  const script = function(props) {
    const myLibOpts = {
      prop1: props.myprop1,
      prop2: props.myprop2,
    };

    console.log('My lib options: ' + JSON.stringify(myLibOpts));
  };

  // Define a new custom component
  editor.Components.addType('comp-with-js', {
    model: {
      defaults: {
        script,
        // Add some style, just to make the component visible
        style: {
          width: '100px',
          height: '100px',
          background: 'red',
        }
      }
    }
  });

  editor.Components.addType('comp-with-js', {
    model: {
      defaults: {
        script,
        // Define default values for your custom properties
        myprop1: 'value1',
        myprop2: '10',
        // Define traits, in order to change your properties
        traits: [
          {
            type: 'select',
            name: 'myprop1',
            changeProp: true,
            options: [
              { value: 'value1', name: 'Value 1' },
              { value: 'value2', name: 'Value 2' },
            ],
          }, {
            type: 'number',
            name: 'myprop2',
            changeProp: true,
          }
        ],
        // Define which properties to pass (this will also reset your script on their changes)
        'script-props': ['myprop1', 'myprop2'],
        style: {
          width: '100px',
          height: '100px',
          background: 'red',
        }
      }
    }
  });

  // Create a block for the component, so we can drop it easily
  editor.Blocks.add('test-block', {
    label: 'Test block',
    attributes: { class: 'fa fa-text' },
    content: { type: 'comp-with-js' },
  });
});

const changeElements = () => {
  const wrapper = editor.DomComponents.getWrapper();
  const myComponent = wrapper.find('button')[0];
  myComponent.components("<s>New Button Name</s>");
  wrapper.find('h1')[0].components("Hello World Rockbot!!!");
}
</script>

<template>
  <div class="panel__top">
    <div class="panel__basic-actions" />
    <div class="panel__devices" />
    <div class="panel__switcher" />
  </div>

  <div class="editor-row">
    <div class="editor-canvas">
      <div id="gjs">
        <h1>Hello World Alexander!</h1>
        <p>Test p</p>
        <button>Test Button</button>
        <br />
        <br />
        <hr />
        <br />
        <textarea>Test Text Area</textarea>
      </div>
    </div>

    <div class="panel__right">
      <div class="layers-container" />
      <div class="styles-container" />
      <div class="traits-container" />
    </div>
  </div>

  <div id="blocks" />
  <button @click="changeElements">Change elements</button>
</template>

<style scoped>
.panel__top {
  padding: 0;
  width: 100%;
  display: flex;
  position: initial;
}

.panel__basic-actions.gjs-pn-panel.gjs-pn-basic-actions, .panel__switcher.gjs-pn-panel {
  display: flex;
}

.panel__basic-actions {
  position: initial;
}

.panel__devices {
  position: initial;
}

.panel__switcher {
  position: initial;
}

#gjs {
  border: none;
}

:deep(.gjs-cv-canvas) {
  top: 0;
  width: 100%;
  height: 100%;
}

:deep(.gjs-block) {
  width: auto;
  height: auto;
  min-height: auto;
}

.gjs-pn-panel {
  display: block;
  position: inherit;
}

.editor-row {
  display: flex;
  justify-content: flex-start;
  align-items: stretch;
  flex-wrap: nowrap;
}

.editor-canvas {
  order: 1;
  flex-grow: 1;
}

.panel__right {
  flex-basis: 230px;
  position: relative;
  overflow-y: auto;
}

:deep(.gjs-pn-buttons) {
  justify-content: flex-start;
}
</style>
