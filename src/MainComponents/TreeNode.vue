<template>
  <!-- <draggable
    class="node"
    tag="ul"
    :list="valuetorender"
    :group="{ name: 'g1' }"
    item-key="label"
  > -->
  <li
    ref="currentNode"
    :class="containerClass"
    role="treeitem"
    :aria-label="label(node)"
    :aria-selected="ariaSelected"
    :aria-expanded="expanded"
    :aria-setsize="node.children ? node.children.length : 0"
    :aria-posinset="index + 1"
    :aria-level="level"
    :aria-checked="ariaChecked"
    :tabindex="index === 0 ? 0 : -1"
    @keydown="onKeyDown"
    class="selected-li"
  >
    <div
      :id="node.key"
      class="node_node"
      :class="contentClass"
      @click="onClick"
      @touchend="onTouchEnd"
      :style="node.style"
    >
      <button
        v-ripple
        type="button"
        class="p-tree-toggler p-link ml-2 mr-3"
        @click="toggle"
        tabindex="-1"
        aria-hidden="true"
      >
        <span :class="toggleIcon"> </span><i class="pi pi-folder ml-1 mr-2"></i>
      </button>
      <div
        v-if="checkboxMode"
        class="p-checkbox p-component"
        aria-hidden="true"
      >
        <div :class="checkboxClass" role="checkbox">
          <span :class="checkboxIcon"></span>
        </div>
      </div>
      <span :class="icon"></span>
      <span class="p-treenode-label">
        <component
          v-if="templates[node.type] || templates['default']"
          :is="templates[node.type] || templates['default']"
          :node="node"
        />
        <template v-else>{{ label(node) }}</template>
      </span>
    </div>
    <span class="icons flex">
      <Button class="px-2 bg-transparent border-0" :disabled="disabled">
        <i class="pi pi-plus-circle"></i
      ></Button>
      <button class="px-2 bg-transparent border-0" :disabled="disabled">
        <i class="pi pi-pencil"></i>
      </button>
      <button class="px-2 bg-transparent border-0" :disabled="disabled">
        <i class="pi pi-trash"></i>
      </button>
      <button @click="disable(node.key)" class="px-2 bg-transparent border-0">
        <i class="pi pi-power-off"></i>
      </button>
    </span>
    <ul
      v-if="hasChildren && expanded && !disabled"
      class="p-treenode-children"
      role="group"
    >
      <TreeNode
        v-for="childNode of node.children"
        :key="childNode.key"
        :node="childNode"
        :templates="templates"
        :level="level + 1"
        :expandedKeys="expandedKeys"
        @node-toggle="onChildNodeToggle"
        @node-click="onChildNodeClick"
        :selectionMode="selectionMode"
        :selectionKeys="selectionKeys"
        @checkbox-change="propagateUp"
      />
    </ul>
  </li>
</template>

<script>
// import { objectExpression } from "@babel/types";
import Ripple from "primevue/ripple";
import { DomHandler } from "primevue/utils";

export default {
  name: "TreeNode",
  emits: ["node-toggle", "node-click", "checkbox-change"],
  props: {
    node: {
      type: null,
      default: null,
    },
    expandedKeys: {
      type: null,
      default: null,
    },
    selectionKeys: {
      type: null,
      default: null,
    },
    selectionMode: {
      type: String,
      default: null,
    },
    templates: {
      type: null,
      default: null,
    },
    level: {
      type: Number,
      default: null,
    },
    index: {
      type: Number,
      default: null,
    },
  },
  nodeTouched: false,
  data() {
    return {
      disabled: false,
      flagColor: false,
      expansionIcon: false,
      boxShadow: false,
    };
  },

  mounted() {
    const hasTreeSelectParent = this.$refs.currentNode.closest(
      ".p-treeselect-items-wrapper"
    );

    if (hasTreeSelectParent) {
      this.setAllNodesTabIndexes();
    }
  },
  methods: {
    selectedDiv(id) {
      let selectedLi = document.querySelectorAll(".node_node")[id];
      //   for (let i = 0; i < selectedLi.length; i++) {
      //   console.log(selectedLi);
      selectedLi.style.border = "1px solid transparent";
      //   }
    },
    disable(id) {
      ///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
      this.disabled = !this.disabled;
      let selectednode = document.querySelectorAll(".node_node")[id];

      this.flagColor
        ? (selectednode.style.backgroundColor = "transparent")
        : (selectednode.style.backgroundColor = "lightgrey");
      this.flagColor = !this.flagColor;

      let catchedBtn = document.getElementsByClassName("p-link")[id];
      this.expansionIcon
        ? (catchedBtn.style.opacity = "1")
        : (catchedBtn.style.opacity = "0");
      this.expansionIcon = !this.expansionIcon;

      //   this.selectionKeys === false;
      //   this.selectionKeys['type']
      //   console.log(Object.entries(this.selectionKeys));
      //   for()
      //   this.selectionKeys =! this.selectionKeys;
    },
    toggle() {
      this.$emit("node-toggle", this.node);
    },
    label(node) {
      return typeof node.label === "function" ? node.label() : node.label;
    },
    onChildNodeToggle(node) {
      this.$emit("node-toggle", node);
    },
    onClick(event) {
      if (!this.disabled) {
        if (
          DomHandler.hasClass(event.target, "p-tree-toggler") ||
          DomHandler.hasClass(event.target.parentElement, "p-tree-toggler")
        ) {
          return;
        }

        if (this.isCheckboxSelectionMode()) {
          this.toggleCheckbox();
        } else {
          this.$emit("node-click", {
            originalEvent: event,
            nodeTouched: this.nodeTouched,
            node: this.node,
          });
        }
        this.nodeTouched = false;
        // this.disabled = true;
      }
      if (this.disabled) {
        console.log("else");
        // console.log(event.target.id);
        // console.log(id);
        let catchedDiv = document.querySelectorAll(".p-treenode-content")[
          event.target.id
        ];
        catchedDiv.style.boxShadow = "inset 0 0 0 0.15rem yellow";
        // console.log(catchedDiv);
        this.disabled =!this.disabled
      }
    },
    onChildNodeClick(event) {
      this.$emit("node-click", event);
    },
    onTouchEnd() {
      this.nodeTouched = true;
    },
    onKeyDown(event) {
      if (!this.isSameNode(event)) return;

      switch (event.code) {
        case "Tab":
          this.onTabKey(event);

          break;

        case "ArrowDown":
          this.onArrowDown(event);

          break;

        case "ArrowUp":
          this.onArrowUp(event);

          break;

        case "ArrowRight":
          this.onArrowRight(event);

          break;

        case "ArrowLeft":
          this.onArrowLeft(event);

          break;

        case "Enter":
        case "Space":
          this.onEnterKey(event);

          break;

        default:
          break;
      }
    },
    onArrowDown(event) {
      const nodeElement = event.target;
      const listElement = nodeElement.children[1];

      if (listElement) {
        this.focusRowChange(nodeElement, listElement.children[0]);
      } else {
        if (nodeElement.nextElementSibling) {
          this.focusRowChange(nodeElement, nodeElement.nextElementSibling);
        } else {
          let nextSiblingAncestor = this.findNextSiblingOfAncestor(nodeElement);

          if (nextSiblingAncestor) {
            this.focusRowChange(nodeElement, nextSiblingAncestor);
          }
        }
      }

      event.preventDefault();
    },
    onArrowUp(event) {
      const nodeElement = event.target;

      if (nodeElement.previousElementSibling) {
        this.focusRowChange(
          nodeElement,
          nodeElement.previousElementSibling,
          this.findLastVisibleDescendant(nodeElement.previousElementSibling)
        );
      } else {
        let parentNodeElement = this.getParentNodeElement(nodeElement);

        if (parentNodeElement) {
          this.focusRowChange(nodeElement, parentNodeElement);
        }
      }

      event.preventDefault();
    },
    onArrowRight(event) {
      if (this.leaf || this.expanded) return;

      event.currentTarget.tabIndex = -1;

      this.$emit("node-toggle", this.node);
      this.$nextTick(() => {
        this.onArrowDown(event);
      });
    },
    onArrowLeft(event) {
      const togglerElement = DomHandler.findSingle(
        event.currentTarget,
        ".p-tree-toggler"
      );

      if (this.level === 0 && !this.expanded) {
        return false;
      }

      if (this.expanded && !this.leaf) {
        togglerElement.click();

        return false;
      }

      const target = this.findBeforeClickableNode(event.currentTarget);

      if (target) {
        this.focusRowChange(event.currentTarget, target);
      }
    },
    onEnterKey(event) {
      this.setTabIndexForSelectionMode(event, this.nodeTouched);
      this.onClick(event);

      event.preventDefault();
    },
    onTabKey() {
      this.setAllNodesTabIndexes();
    },
    setAllNodesTabIndexes() {
      const nodes = DomHandler.find(
        this.$refs.currentNode.closest(".p-tree-container"),
        ".p-treenode"
      );

      const hasSelectedNode = [...nodes].some(
        (node) =>
          node.getAttribute("aria-selected") === "true" ||
          node.getAttribute("aria-checked") === "true"
      );

      [...nodes].forEach((node) => {
        node.tabIndex = -1;
      });

      if (hasSelectedNode) {
        const selectedNodes = [...nodes].filter(
          (node) =>
            node.getAttribute("aria-selected") === "true" ||
            node.getAttribute("aria-checked") === "true"
        );

        selectedNodes[0].tabIndex = 0;

        return;
      }

      [...nodes][0].tabIndex = 0;
    },
    setTabIndexForSelectionMode(event, nodeTouched) {
      if (this.selectionMode !== null) {
        const elements = [
          ...DomHandler.find(
            this.$refs.currentNode.parentElement,
            ".p-treenode"
          ),
        ];

        event.currentTarget.tabIndex = nodeTouched === false ? -1 : 0;

        if (elements.every((element) => element.tabIndex === -1)) {
          elements[0].tabIndex = 0;
        }
      }
    },
    focusRowChange(
      firstFocusableRow,
      currentFocusedRow,
      lastVisibleDescendant
    ) {
      firstFocusableRow.tabIndex = "-1";
      currentFocusedRow.tabIndex = "0";

      this.focusNode(lastVisibleDescendant || currentFocusedRow);
    },
    findBeforeClickableNode(node) {
      const parentListElement = node.closest("ul").closest("li");

      if (parentListElement) {
        const prevNodeButton = DomHandler.findSingle(
          parentListElement,
          "button"
        );

        if (prevNodeButton && prevNodeButton.style.visibility !== "hidden") {
          return parentListElement;
        }

        return this.findBeforeClickableNode(node.previousElementSibling);
      }

      return null;
    },
    toggleCheckbox() {
      let _selectionKeys = this.selectionKeys ? { ...this.selectionKeys } : {};
      const _check = !this.checked;

      this.propagateDown(this.node, _check, _selectionKeys);

      this.$emit("checkbox-change", {
        node: this.node,
        check: _check,
        selectionKeys: _selectionKeys,
      });
    },
    propagateDown(node, check, selectionKeys) {
      if (check)
        selectionKeys[node.key] = { checked: true, partialChecked: false };
      else delete selectionKeys[node.key];

      if (node.children && node.children.length) {
        for (let child of node.children) {
          this.propagateDown(child, check, selectionKeys);
        }
      }
    },
    propagateUp(event) {
      let check = event.check;
      let _selectionKeys = { ...event.selectionKeys };
      let checkedChildCount = 0;
      let childPartialSelected = false;

      for (let child of this.node.children) {
        if (_selectionKeys[child.key] && _selectionKeys[child.key].checked)
          checkedChildCount++;
        else if (
          _selectionKeys[child.key] &&
          _selectionKeys[child.key].partialChecked
        )
          childPartialSelected = true;
      }

      if (check && checkedChildCount === this.node.children.length) {
        _selectionKeys[this.node.key] = {
          checked: true,
          partialChecked: false,
        };
      } else {
        if (!check) {
          delete _selectionKeys[this.node.key];
        }

        if (
          childPartialSelected ||
          (checkedChildCount > 0 &&
            checkedChildCount !== this.node.children.length)
        )
          _selectionKeys[this.node.key] = {
            checked: false,
            partialChecked: true,
          };
        else delete _selectionKeys[this.node.key];
      }

      this.$emit("checkbox-change", {
        node: event.node,
        check: event.check,
        selectionKeys: _selectionKeys,
      });
    },
    onChildCheckboxChange(event) {
      this.$emit("checkbox-change", event);
    },
    findNextSiblingOfAncestor(nodeElement) {
      let parentNodeElement = this.getParentNodeElement(nodeElement);

      if (parentNodeElement) {
        if (parentNodeElement.nextElementSibling)
          return parentNodeElement.nextElementSibling;
        else return this.findNextSiblingOfAncestor(parentNodeElement);
      } else {
        return null;
      }
    },
    findLastVisibleDescendant(nodeElement) {
      const childrenListElement = nodeElement.children[1];

      if (childrenListElement) {
        const lastChildElement =
          childrenListElement.children[childrenListElement.children.length - 1];

        return this.findLastVisibleDescendant(lastChildElement);
      } else {
        return nodeElement;
      }
    },
    getParentNodeElement(nodeElement) {
      const parentNodeElement = nodeElement.parentElement.parentElement;

      return DomHandler.hasClass(parentNodeElement, "p-treenode")
        ? parentNodeElement
        : null;
    },
    focusNode(element) {
      element.focus();
    },
    isCheckboxSelectionMode() {
      return this.selectionMode === "checkbox";
    },
    isSameNode(event) {
      return (
        event.currentTarget &&
        (event.currentTarget.isSameNode(event.target) ||
          event.currentTarget.isSameNode(event.target.closest(".p-treenode")))
      );
    },
  },
  computed: {
    hasChildren() {
      return this.node.children && this.node.children.length > 0;
    },
    expanded() {
      return this.expandedKeys && this.expandedKeys[this.node.key] === true;
    },
    leaf() {
      return this.node.leaf === false
        ? false
        : !(this.node.children && this.node.children.length);
    },
    selectable() {
      return this.node.selectable === false
        ? false
        : this.selectionMode != null;
    },
    selected() {
      return this.selectionMode && this.selectionKeys
        ? this.selectionKeys[this.node.key] === true
        : false;
    },
    containerClass() {
      return ["p-treenode", { "p-treenode-leaf": this.leaf }];
    },
    contentClass() {
      return [
        "p-treenode-content",
        this.node.styleClass,
        {
          "p-treenode-selectable": this.selectable,
          "p-highlight": this.checkboxMode ? this.checked : this.selected,
        },
      ];
    },
    icon() {
      return ["p-treenode-icon", this.node.icon];
    },
    toggleIcon() {
      return [
        "p-tree-toggler-icon pi pi-fw",
        this.expanded && !this.disabled
          ? this.node.expandedIcon || "pi-chevron-down"
          : this.node.collapsedIcon || "pi-chevron-right",
      ];
    },
    checkboxClass() {
      return [
        "p-checkbox-box",
        { "p-highlight": this.checked, "p-indeterminate": this.partialChecked },
      ];
    },
    checkboxIcon() {
      return [
        "p-checkbox-icon",
        { "pi pi-check": this.checked, "pi pi-minus": this.partialChecked },
      ];
    },
    checkboxMode() {
      return (
        this.selectionMode === "checkbox" && this.node.selectable !== false
      );
    },
    checked() {
      return this.selectionKeys
        ? this.selectionKeys[this.node.key] &&
            this.selectionKeys[this.node.key].checked
        : false;
    },
    partialChecked() {
      return this.selectionKeys
        ? this.selectionKeys[this.node.key] &&
            this.selectionKeys[this.node.key].partialChecked
        : false;
    },
    ariaChecked() {
      return this.selectionMode === "single" ||
        this.selectionMode === "multiple"
        ? this.selected
        : undefined;
    },
    ariaSelected() {
      return this.checkboxMode ? this.checked : undefined;
    },
  },
  directives: {
    ripple: Ripple,
  },
};
</script>
<style scoped>
.p-tree-toggler {
  width: 3em !important;
  height: 3em !important;
  color: black !important;
}
span.icons {
  margin-left: auto;
}
.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight {
  background-color: transparent;
  color: black !important;
  box-shadow: inset 0 0 0 0.15rem green;
}
.p-tree .p-tree-container .p-treenode:focus > .p-treenode-content {
  /* box-shadow: inset 0 0 0 0.15rem red; */
}
/* .node_node {
    position: relative;
} */
.selected-li {
  position: relative;
}
.icons {
  position: absolute;
  z-index: 20;
  right: 2em;
  top: 1em;
}
</style>
