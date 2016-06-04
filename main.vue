<template>
  <div class="vuelation-affix" :style="affixedStyles">
    <slot></slot>
  </div>
</template>

<script>
  import debounce from 'lodash.debounce';
  import 'javascript-detect-element-resize';

  export default {
    beforeDestroy() {
      removeResizeListener(this.$el.parentElement, this.parentResizing);
      window.removeEventListener('resize', this.windowResizing);
      window.removeEventListener('scroll', this.scrolling);
    },

    data() {
      return {
        affixed:         false,
        affixedStyles:   { position: 'relative' },
        affixedToBottom: false,
        elementSizes:    {},
        parentSizes:     {},
        windowSizes:     {}
      };
    },

    methods: {
      affixElement() {
        let element = this.$el;
        this.affixedStyles = {
          bottom:   this.elementSizes.bottom,
          height:   element.offsetHeight,
          left:     this.getOffset(element).left,
          top:      0,
          width:    element.offsetWidth,
          position: 'fixed'
        };
      },

      affixElementToBottom() {
        this.affixedStyles = {
          bottom:   0,
          height:   this.elementSizes.height,
          left:     this.elementSizes.left,
          position: 'absolute',
          width:    this.elementSizes.width
        };
      },

      elementResizing() {
        let element = this.$el;
        this.elementSizes = {
          outerHeight: element.offsetHeight,
          offsetTop:   this.getOffset(element).top
        };
        this.elementSizes.offsetBottom = this.elementSizes.outerHeight + this.elementSizes.offsetTop;
      },

      elementShouldBeAtBottom(windowScrollTop) {
        var elementBottom = windowScrollTop + this.elementSizes.outerHeight;
        return elementBottom >= this.parentSizes.offsetBottom;
      },

      elementShouldBeAtTop(windowScrollTop) {
        return windowScrollTop <= this.elementSizes.offsetTop;
      },

      elementShouldBeFixed(windowScrollTop) {
        return windowScrollTop > this.elementSizes.offsetTop;
      },

      getOffset(element) {
        const rect = element.getBoundingClientRect();
        const body = document.body;
        const clientTop = element.clientTop || body.clientTop || 0;
        const clientLeft = element.clientLeft || body.clientLeft || 0;
        const scrollTop = this.getScroll(window, true);
        const scrollLeft = this.getScroll(window);

        return {
          top: rect.top + scrollTop - clientTop,
          left: rect.left + scrollLeft - clientLeft
        };
      },

      getScroll(w, top) {
        let ret = w['page' + (top ? 'Y' : 'X') + 'Offset'];
        const method = 'scroll' + (top ? 'Top' : 'Left');

        if (typeof ret !== 'number') {
          const d = w.document;
          // ie6,7,8 standard mode
          ret = d.documentElement[method];

          if (typeof ret !== 'number') {
            // quirks mode
            ret = d.body[method];
          }
        }
        return ret;
      },

      parentResizing() {
        let parent = this.$el.parentElement;

        this.parentSizes = {
          outerHeight: parent.offsetHeight,
          offsetTop:   this.getOffset(parent).top
        };
        this.parentSizes.offsetBottom = this.parentSizes.outerHeight + this.parentSizes.offsetTop;
      },

      scrolling() {
        let element = this.$el;
        let parent = element.parentElement;
        let windowScrollTop = this.getScroll(window, true);

        if (
          window.innerHeight <= this.elementSizes.outerHeight           ||
          this.parentSizes.outerHeight <= this.elementSizes.outerHeight ||
          (this.affixed && this.elementShouldBeAtTop(windowScrollTop))
        ) {
          this.affixed = false;
          this.affixedToBottom = false;
          this.unaffixElement();
        } else if (this.elementShouldBeAtBottom(windowScrollTop)) {
          this.affixed = false;
          this.affixedToBottom = true;
          this.affixElementToBottom();
        } else if (!this.affixed && this.elementShouldBeFixed(windowScrollTop)) {
          this.affixed = true;
          this.affixedToBottom = false;
          this.affixElement();
        }
      },

      unaffixElement() {
        this.affixedStyles = { position: 'relative' };
      },

      windowResizing() {
        this.elementResizing();
        this.parentResizing();
      }
    },

    props: {
      disabled: {
        default: false,
        type:    Boolean
      }
    },

    ready() {
      // Set original position for the parent element to relative
      this.$el.parentElement.style.position = 'relative';

      // Save the sizes and positions of the affixed element and parent element
      this.elementResizing();
      this.parentResizing();

      // Add event listeners for scrolling and window/element resizing
      addResizeListener(this.$el.parentElement, this.parentResizing);
      window.addEventListener('resize', debounce(this.windowResizing, 200));
      window.addEventListener('scroll', this.scrolling);
    },

    watch: {
      'disabled': function(value) {
        if (value) {
          removeResizeListener(this.$el.parentElement, this.parentResizing);
          window.removeEventListener('resize', this.windowResizing);
          window.removeEventListener('scroll', this.scrolling);
          this.affixed = false;
          this.affixedToBottom = false;
          this.unaffixElement();
        } else {
          addResizeListener(this.$el.parentElement, this.parentResizing);
          window.addEventListener('resize', debounce(this.windowResizing, 200));
          window.addEventListener('scroll', this.scrolling);
        }
      },

      'parentSizes.outerHeight': 'scrolling'
    }
  };
</script>
