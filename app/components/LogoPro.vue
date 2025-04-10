<template>
  <div class="flex items-center" role="img" aria-labelledby="rbg-logo">
    <svg
      id="routiine_logo"
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
      viewBox="0 0 48 48"
      class="w-[48px] h-[48px]"
    >
      <title id="rbg-logo">Routiine LLC</title>
      <defs>
        <radialGradient
          id="radial-gradient"
          cx="24"
          cy="24"
          fx="-3.01"
          fy="24"
          r="27.01"
          gradientUnits="userSpaceOnUse"
        >
          <stop offset=".11" stop-color="#29a3dc" />
          <stop offset=".65" stop-color="#006aa2" />
          <stop offset=".88" stop-color="#083450" />
          <stop offset="1" stop-color="#062f49" />
        </radialGradient>
      </defs>
      <path
        d="M26.52,36.09c-.29-.28-.23-.7,0-.88.25-.2.63-.18.87.06l2.9,2.67c1.35,1.25,2.36.58,2.65.33..."
        fill="url(#radial-gradient)"
      />
    </svg>
    <h2
      class="font-satoshi-800 hidden md:flex h-12 items-center text-[20px] md:text-[28px] tracking-tight ml-2"
    >
      Routiine.io
    </h2>
  </div>
</template>
