const PLUGIN_NAME = 'growi-plugin-editor-toolbar-top';
const STATE_KEY = '__growiEditorToolbarTopPlugin';
const TOOLBAR_MARK = 'data-growi-toolbar-top-active';

const isVisible = (element) => {
  if (!(element instanceof HTMLElement)) return false;

  const rect = element.getBoundingClientRect();
  const style = window.getComputedStyle(element);

  return rect.width > 0
    && rect.height > 0
    && style.display !== 'none'
    && style.visibility !== 'hidden';
};

const findEditor = () => {
  return [...document.querySelectorAll('.cm-editor')].find((element) => {
    return isVisible(element)
      && element.querySelector(
        '.cm-content[contenteditable="true"][data-language="markdown"]',
      ) != null;
  }) ?? null;
};

const hasAddButton = (element) => {
  return [...element.querySelectorAll('button .material-symbols-outlined')]
    .some((icon) => icon.textContent?.trim() === 'add');
};

const findToolbar = (editor) => {
  const templateButton = [
    ...document.querySelectorAll(
      'button[data-testid="open-template-button"]',
    ),
  ].find(isVisible);

  if (templateButton == null) return null;

  const innerToolbar = templateButton.closest('div.d-flex.gap-2');
  if (!(innerToolbar instanceof HTMLElement)) return null;

  // 「＋」ボタンが同じ列にある場合は、その列を含む最小の親まで広げる。
  let candidate = innerToolbar;
  let current = innerToolbar.parentElement;

  for (let depth = 0; depth < 5 && current != null; depth += 1) {
    if (current.contains(editor)) break;

    const rect = current.getBoundingClientRect();
    if (hasAddButton(current) && rect.height > 0 && rect.height <= 100) {
      candidate = current;
      break;
    }

    current = current.parentElement;
  }

  return candidate;
};

const rememberStyle = (state, element) => {
  if (!state.originalStyles.has(element)) {
    state.originalStyles.set(element, {
      hadStyle: element.hasAttribute('style'),
      cssText: element.getAttribute('style') ?? '',
    });
  }
};

const restoreStyles = (state) => {
  for (const [element, original] of state.originalStyles.entries()) {
    if (!element.isConnected) continue;

    if (original.hadStyle) {
      element.setAttribute('style', original.cssText);
    }
    else {
      element.removeAttribute('style');
    }

    element.removeAttribute(TOOLBAR_MARK);
  }

  state.originalStyles.clear();
};

const setImportant = (element, property, value) => {
  element.style.setProperty(property, value, 'important');
};

const applyToolbarPosition = (state) => {
  state.timerId = 0;

  const editor = findEditor();
  if (editor == null) return;

  const toolbar = findToolbar(editor);
  if (toolbar == null) return;

  const editorRect = editor.getBoundingClientRect();
  const measuredHeight = toolbar.getBoundingClientRect().height;
  const toolbarHeight = Math.max(40, Math.ceil(measuredHeight || 0));

  rememberStyle(state, editor);
  rememberStyle(state, toolbar);

  // DOMを移動せず、実物のツールバーを編集欄上端へ固定表示する。
  // Reactの再描画で元の位置へ戻される問題を避けるための方式。
  const top = Math.max(0, Math.min(
    editorRect.top,
    editorRect.bottom - toolbarHeight,
  ));

  const isEditorOnScreen = editorRect.bottom > 0
    && editorRect.top < window.innerHeight;

  setImportant(toolbar, 'position', 'fixed');
  setImportant(toolbar, 'top', `${Math.round(top)}px`);
  setImportant(toolbar, 'right', 'auto');
  setImportant(toolbar, 'bottom', 'auto');
  setImportant(toolbar, 'left', `${Math.round(editorRect.left)}px`);
  setImportant(toolbar, 'width', `${Math.round(editorRect.width)}px`);
  setImportant(toolbar, 'min-height', `${toolbarHeight}px`);
  setImportant(toolbar, 'box-sizing', 'border-box');
  setImportant(toolbar, 'display', isEditorOnScreen ? 'flex' : 'none');
  setImportant(toolbar, 'align-items', 'center');
  setImportant(toolbar, 'justify-content', 'flex-start');
  setImportant(toolbar, 'overflow-x', 'auto');
  setImportant(toolbar, 'overflow-y', 'hidden');
  setImportant(toolbar, 'margin', '0');
  setImportant(toolbar, 'padding', '2px 8px');
  setImportant(toolbar, 'z-index', '1100');
  setImportant(toolbar, 'background', 'var(--bs-body-bg, #172331)');
  setImportant(toolbar, 'border-top', 'none');
  setImportant(
    toolbar,
    'border-bottom',
    '1px solid var(--bs-border-color, #495057)',
  );
  setImportant(toolbar, 'transform', 'none');

  // ツールバーが1行目へ重ならないよう、エディター上部に空間を確保する。
  setImportant(editor, 'box-sizing', 'border-box');
  setImportant(editor, 'padding-top', `${toolbarHeight}px`);

  toolbar.setAttribute(TOOLBAR_MARK, 'true');
};

const scheduleApply = (state) => {
  if (!state.active) return;

  if (state.timerId !== 0) {
    window.clearTimeout(state.timerId);
  }

  state.timerId = window.setTimeout(
    () => applyToolbarPosition(state),
    80,
  );
};

const destroyState = (state) => {
  if (state == null || !state.active) return;

  state.active = false;
  state.observer?.disconnect();

  if (state.timerId !== 0) {
    window.clearTimeout(state.timerId);
  }

  window.removeEventListener('resize', state.onViewportChange);
  window.removeEventListener('scroll', state.onViewportChange, true);
  window.removeEventListener('popstate', state.onViewportChange);

  restoreStyles(state);
};

const activate = () => {
  const previousState = window[STATE_KEY];
  if (previousState != null) {
    destroyState(previousState);
  }

  const state = {
    active: true,
    timerId: 0,
    observer: null,
    originalStyles: new Map(),
    onViewportChange: null,
  };

  state.onViewportChange = () => scheduleApply(state);

  state.observer = new MutationObserver(() => scheduleApply(state));
  state.observer.observe(document.documentElement, {
    childList: true,
    subtree: true,
  });

  window.addEventListener('resize', state.onViewportChange);
  window.addEventListener('scroll', state.onViewportChange, true);
  window.addEventListener('popstate', state.onViewportChange);

  window[STATE_KEY] = state;
  scheduleApply(state);
};

const deactivate = () => {
  const state = window[STATE_KEY];
  destroyState(state);
  delete window[STATE_KEY];
};

if (window.pluginActivators == null) {
  window.pluginActivators = {};
}

window.pluginActivators[PLUGIN_NAME] = {
  activate,
  deactivate,
};
