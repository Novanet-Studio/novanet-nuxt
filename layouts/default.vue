<template>
  <Transition name="fade" appear>
    <div class="layout">
      <div class="container">
        <app-header />
        <main class="main">
          <slot />
        </main>
        <app-footer />
      </div>
    </div>
  </Transition>
</template>
