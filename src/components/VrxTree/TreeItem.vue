<template>
  <div data-testid="vrx-tree-node" class="w-auto h-full flex flex-col" :class="isParent ? null : 'pl-5'" @click.stop="() => cellClicked(node, props.parentId)" ref="elementRef" >
    <div class="tree-element vrxtree-element-style rounded-s" >
      <VrxIcon :icon="node.children.length > 0 ? 'chevron-right': 'empty'" :class="open ? 'icon-rotate' : 'icon-off'" size="5" @click="clickHandle" />
      <VrxIcon v-if="node.icon" :icon="node.icon" size="4"/>
      <input
          data-testid="vrx-tree-node-checkbox"
          v-if="selectable"
          type="checkbox"
          class="form-checkbox h-4 w-4"
          v-model="checkValue"
          @click="selectHandle"
          :indeterminate.prop="hasChildrenChecked && !checkValue"
      />
      <div class="w-full flex justify-between items-center flex-row">
        <component v-if="node.asComponent === true && node.componentProps" :is="node.component" v-bind="node.componentProps()">
          {{ node.componentSlots ?? node.text }}
        </component>
        <span v-else  :class="node.class">{{ props.node.text }}</span>
        <component v-if="node.rightComponent && node.rightComponentProps" :is="node.rightComponent" v-bind="node.rightComponentProps()" />
      </div>

    </div>


    <!-- CHILDREN RECURSIVE -->
    <TreeItem
        v-if="node.children.length > 0 && open"
        v-for="child in node.children"
        :node="child"
        :key="child.id"
        :selectable="selectable"
        :selected="checkValue"
        :selected-nodes="selectedNodes"
        :add-node="addNode"
        :remove-node="removeNode"
        :parent-id="props.node.id"
        :siblings="node.children"
        :remove-node-by-id="removeNodeById"
        @check-clicked="checkClicked"
        @cellClicked="(value, parentId, element) => cellClicked(value, parentId, element)"
    />

  </div>
</template>

<script setup lang="ts">
  import VrxIcon from "@/components/VrxIcon/VrxIcon.vue";
  import {VrxTreeNode} from "@/components/VrxTree/VrxTree.types.ts";
  import {Ref, ref, watch} from "vue";

  const props = defineProps<{
    node: VrxTreeNode,
    selectable: boolean,
    isParent?: boolean,
    selected?: boolean,
    selectedNodes: string[],
    addNode: (nodeId: string) => void
    removeNodeById: (nodeId: string, isParent?: boolean) => void
    removeNode: (node: VrxTreeNode) => void
    parentId: string,
    siblings: VrxTreeNode[]
  }>();

  const elementRef = ref<Element | null>(null);

  const open = ref(props.node.open);
  const checkValue : Ref<boolean>= ref(props.selected || props.selectedNodes.includes(props.parentId));
  const hasChildrenChecked = ref(false);

  watch(() => props.selectedNodes,(newValue) => {
    checkValue.value = newValue.includes(props.node.id) || newValue.includes(props.parentId) || props.selected;
    hasChildrenChecked.value = newValue.filter((node : string) => node.includes(props.node.id)).length > 0;
  },{immediate: true, deep: true});

  watch(() => props.selected,(newValue) => {
    checkValue.value = newValue || props.selectedNodes.includes(props.node.id);
  },{immediate: true, deep: true});

  /**
   * Handle click for inputs and tree elements
   * @param event
   */
  const clickHandle = (event: MouseEvent) => {
    // @ts-ignore
    if(event.target.nodeName !== "INPUT"){
      open.value = !open.value;
    }
  }

  const cellClicked = (value : VrxTreeNode, parentId : string, element ?: Element ) => {

    emit('cellClicked', value, parentId, element ? element : elementRef.value);
  }

  /**
   * Handle the click on the checkbox
   */
  const selectHandle = () => {
    checkValue.value = !checkValue.value;
    checkValue.value ? props.addNode(props.node.id) : props.removeNodeById(props.node.id, props.isParent);

    if(props.node.children.length > 0){
      props.node.children.forEach((child : VrxTreeNode) => {
        props.removeNode(child);
      })
    }

    if(!props.isParent){
      checkParent();
      checkSiblingsAndParent();
      emit('checkClicked', props.node);
    }
  }


  /**
   * Check if all siblings are selected and if so, add the parent
   */
  const checkSiblingsAndParent = () => {
    let all = true;
    props.siblings.forEach((node : VrxTreeNode) => {
      if(!props.selectedNodes.includes(node.id)){
        all = false;
      }
    });
    if(all){
      props.siblings.forEach((node : VrxTreeNode) => {
        props.removeNodeById(node.id);
      })
      props.addNode(props.parentId);
    }
  }

  /**
   * Check if parent is selected and if so, remove it and add all siblings
   */
  const checkParent = () => {
    // If parent is actually selected, remove it from selected nodes and add all siblings
    if(props.selectedNodes.includes(props.parentId) || props.selected){
      props.removeNodeById(props.parentId, props.isParent);
      props.siblings.forEach((sibling : VrxTreeNode) => {
        if(sibling.id !== props.node.id)
          props.addNode(sibling.id);
      })
    }
  }

  /**
   * Emitted when a children checkbox is clicked
   */
  const checkClicked = () => {
    if(props.isParent) return;
    checkParent();
    checkSiblingsAndParent();
    emit('checkClicked', props.node);
  }

  const emit = defineEmits(['checkClicked', 'cellClicked']);

</script>


<style scoped>
  .tree-element{
    display: flex;
    flex-direction: row;
    align-items: center;
    cursor: pointer;
    gap: 5px;
  }

</style>
