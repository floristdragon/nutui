<template>
  <Teleport :to="teleport">
    <nut-overlay
      :show="show && overlay"
      :class="overlayClass"
      :style="overlayStyle"
      :zIndex="state.zIndex"
      :duration="duration"
      @click="onClickOverlay"
    />
    <Transition
      :name="state.transitionName"
      @after-enter="onOpened"
      @after-leave="onClosed"
    >
      <view v-show="show" :class="classes" :style="popStyle" @click="onClick">
        <slot v-if="state.showSlot"></slot>
        <nut-icon
          v-if="closeable"
          @click="onClickCloseIcon"
          :name="closeIcon"
          size="12px"
          class="nutui-popup__close-icon"
          :class="'nutui-popup__close-icon--' + closeIconPosition"
        >
        </nut-icon>
      </view>
    </Transition>
  </Teleport>
</template>
<script lang="ts">
import {
  onMounted,
  onBeforeMount,
  onBeforeUnmount,
  onActivated,
  onDeactivated,
  watch,
  computed,
  reactive,
  PropType,
  CSSProperties
} from 'vue';
import { useLockScroll } from './use-lock-scroll';
import { overlayProps } from './../overlay/index.vue';
import { createComponent } from '@/utils/create';
const { componentName, create } = createComponent('popup');

// let _global = {
//   zIndex: 2000,
//   overLayCount: 0
// };

let _zIndex = 2000;

const popupProps = {
  id: {
    type: [String, Number]
  },
  position: {
    type: String,
    default: 'center'
  },

  transition: String,

  style: {
    type: Object as PropType<CSSProperties>
  },

  popClass: {
    type: String,
    default: ''
  },

  closeable: {
    type: Boolean,
    default: false
  },

  closeIconPosition: {
    type: String,
    default: 'top-right'
  },

  closeIcon: {
    type: String,
    default: 'close'
  },

  destroyOnClose: {
    type: Boolean,
    default: true
  },

  teleport: {
    type: [String, Element],
    default: 'body'
  },

  round: {
    type: Boolean,
    default: false
  }
};

export default create({
  props: {
    ...overlayProps,
    ...popupProps
  },
  emits: [
    'click',
    'click-close-icon',
    'open',
    'close',
    'opend',
    'closed',
    'update:show',
    'click-overlay'
  ],

  setup(props, { emit }) {
    // if (props.id) console.log(props.id);
    let opened;
    let keepAlive;
    // let ocount = 1;
    const state = reactive({
      zIndex: 0,
      showSlot: true,
      transitionName: `popup-fade-${props.position}`,
      overLayCount: 1
    });

    const [lockScroll, unlockScroll] = useLockScroll(() => props.lockScroll);

    const classes = computed(() => {
      const prefixCls = componentName;
      return {
        [prefixCls]: true,
        ['round']: props.round,
        [`popup-${props.position}`]: true,
        [props.popClass]: true
      };
    });

    const popStyle = computed(() => {
      return {
        zIndex: state.zIndex,
        animationDuration: props.duration ? `${props.duration}s` : 'initial',
        ...props.style
      };
    });

    const open = () => {
      if (!opened) {
        if (props.zIndex !== undefined) {
          _zIndex = Number(props.zIndex);
        }
        opened = true;
        lockScroll();
        state.zIndex = ++_zIndex;

        if (props.destroyOnClose) {
          state.showSlot = true;
        }
      }
    };

    const close = () => {
      if (opened) {
        opened = false;
        unlockScroll();
        emit('update:show', false);
        if (props.destroyOnClose) {
          setTimeout(() => {
            state.showSlot = false;
          }, +props.duration * 1000);
        }
      }
    };

    const onClick = e => {
      emit('click', e);
    };

    const onClickCloseIcon = e => {
      emit('click-close-icon', e);
      close();
    };

    const onClickOverlay = e => {
      if (props.closeOnClickOverlay) {
        emit('click-overlay', e);
        close();
      }
    };

    const onOpened = e => {
      emit('opend', e);
    };

    const onClosed = e => {
      emit('closed', e);
    };

    onMounted(() => {
      // console.log(props.id);
      props.transition
        ? (state.transitionName = props.transition)
        : (state.transitionName = `popup-slide-${props.position}`);

      props.show && open();
    });

    onBeforeUnmount(() => {
      props.show && close();
    });

    onBeforeMount(() => {
      if (opened) {
        unlockScroll();
      }
    });

    onActivated(() => {
      if (keepAlive) {
        emit('update:show', true);
        keepAlive = false;
      }
    });

    onDeactivated(() => {
      if (props.show) {
        close();
        keepAlive = true;
      }
    });

    watch(
      () => props.show,
      value => {
        if (value) {
          open();
          emit('open');
        } else {
          close();
          emit('close');
        }
      }
    );

    watch(
      () => props.position,
      value => {
        value === 'center'
          ? (state.transitionName = 'popup-fade')
          : (state.transitionName = `popup-slide-${value}`);
      }
    );

    return {
      onClick,
      onClickCloseIcon,
      onClickOverlay,
      onOpened,
      onClosed,
      state,
      popStyle,
      classes
    };
  }
});
</script>

<style lang="scss">
@import 'index.scss';
</style>
