<script>
  import { onMount } from 'svelte';
  import { isWindows, isLinux } from '../utils/platform.js';
  
  // Copy function
  function copyToClipboard(text) {
    navigator.clipboard.writeText(text).then(() => {
      console.log('Text copied to clipboard');
    }).catch(err => {
      console.error('Failed to copy text: ', err);
    });
  }
  
  const version = import.meta.env.VITE_PUBLIC_GIT_SHA || 'dev'

  let FlashComponent = null;
  let loading = true;
  let error = null;

  onMount(async()  => {
   try {
      const flash = await import('./flash.svelte');
      FlashComponent = flash.default;
   }
   catch(err) {
      error = err;
   }
   finally {
      loading = false;
   }
  });
</script>

<div class="flex flex-col lg:flex-row flex-wrap">
  <main class="p-12 md:p-16 lg:p-20 xl:p-24 w-screen max-w-none lg:max-w-prose lg:w-auto h-auto lg:h-screen lg:overflow-y-auto prose dark:prose-invert prose-green bg-white dark:bg-gray-900">
    <section>
      <img src="/src/assets/comma.svg" alt="comma" width="128" height="128" class="dark:invert" loading="lazy"/>
      <h1>flash.comma.ai</h1>
      <p>
        This tool allows you to flash AGNOS onto your comma device. AGNOS is the Ubuntu-based operating system for
        your <a href="https://comma.ai/shop/comma-3x" target="_blank">comma 3/3X</a>.
      </p>
    </section>
    <hr />

    <section>
      <h2>Requirements</h2>
      <ul>
        <li>
          A web browser which supports <a href="https://caniuse.com/webusb" target="_blank">WebUSB</a>
          (such as Google Chrome, Microsoft Edge, Opera), running on Windows, macOS, Linux, or Android.
        </li>
        <li>
          A good quality USB-C cable to connect the device to your computer. <span title="SuperSpeed">USB 3</span>
          is recommended for faster flashing speed.
        </li>
        <li>
          Another USB-C cable and a charger, to power the device outside your car.
        </li>
      </ul>
      
      {#if isWindows}
        <h3>USB Driver</h3>
        <p>You need additional driver software for Windows before you connect your device.</p>
        <ol>
          <li>
            Download and run <a href="https://zadig.akeo.ie/" target="_blank">Zadig</a>.
          </li>
          <li>
            Under <code>Device</code> in the menu bar, select <code>Create New Device</code>.
            <img 
              src="/src/assets/zadig_create_new_device.png" 
              alt="Zadig Create New Device" 
              width="575" 
              height="254"
              loading="lazy"
            />
          </li>
          <li>
            Fill in three fields. The first field is just a description and you can fill in anything. The next two
            fields are very important. Fill them in with <code>05C6</code> and <code>9008</code>
            respectively. Press "Install Driver" and give it a few minutes to install.
            <img
              src="/src/assets/zadig_form.png"
              alt="Zadig Form"
              width="575"
              height="254"
              loading="lazy"
            />
          </li>
        </ol>
        <p>No additional software is required for macOS, Linux or Android.</p>
      {/if}
    </section>
    <hr />

    <section>
      <h2>Flashing</h2>
      <p>Follow these steps to put your device into QDL mode:</p>
      <ol>
        <li>Unplug the device and wait for the LED to switch off.</li>
        <li>First, connect the device to your computer using the <strong>lower</strong> <span class="whitespace-nowrap">USB-C</span> port <strong>(port 1)</strong>.</li>
        <li>Second, connect power to the <strong>upper</strong> <span class="whitespace-nowrap">OBD-C</span> port <strong>(port 2)</strong>.</li>
      </ol>
      <img
        src="/src/assets/qdl-ports.svg"
        alt="image showing comma three and two ports. the lower port is labeled 1. the upper port is labeled 2."
        width="450"
        height="300"
        loading="lazy"
      />
      <p>Your device's screen will remain blank for the entire flashing process. This is normal.</p>
      
      {#if isLinux}
        <strong>Note for Linux users</strong>
        <p>
          On Linux systems, devices in QDL mode are automatically bound to the kernel's qcserial driver, and
          need to be unbound before we can access the device. Copy the script below into your terminal and run it
          after plugging in your device.
        </p>
        <div class="relative">
          <pre class="font-mono pt-12">for d in /sys/bus/usb/drivers/qcserial/*-*; do [ -e "$d" ] && echo -n "$(basename $d)" | sudo tee /sys/bus/usb/drivers/qcserial/unbind > /dev/null; done</pre>
          <button
            class="absolute top-2 right-2 bg-blue-600 hover:bg-blue-500 active:bg-blue-300 transition-colors text-white p-1 rounded-md"
            on:click={() => copyToClipboard('for d in /sys/bus/usb/drivers/qcserial/*-*; do [ -e "$d" ] && echo -n "$(basename $d)" | sudo tee /sys/bus/usb/drivers/qcserial/unbind > /dev/null; done')}
          >
            Copy
          </button>
        </div>
      {/if}
      
      <p>
        Next, click the button to start flashing. From the prompt select the device which starts with
        "QUSB_BULK".
      </p>
      <p>
        The process can take 30+ minutes depending on your internet connection and system performance. Do not
        unplug the device until all steps are complete.
      </p>
    </section>
    <hr />

    <section>
      <h2>Troubleshooting</h2>
      <h3>Lost connection</h3>
      <p>
        Try using high quality USB 3 cables. You should also try different USB ports on the front or back of your
        computer. If you're using a USB hub, try connecting directly to your computer instead.
      </p>
      <h3>My device's screen is blank</h3>
      <p>
        This is normal in QDL mode. You can verify that the "QUSB_BULK" device shows up when you press
        the Flash button to know that it is working correctly.
      </p>
      <h3>My device says "fastboot mode"</h3>
      <p>
        You may have followed outdated instructions for flashing. Please read the instructions above for putting
        your device into QDL mode.
      </p>
      <h3>General Tips</h3>
      <ul>
        <li>Try another computer or OS</li>
        <li>Try different USB ports on your computer</li>
        <li>Try different USB-C cables; low quality cables are often the source of problems. Note that the included OBD-C cable will not work.</li>
      </ul>
      <h3>Other questions</h3>
      <p>
        If you need help, join our <a href="https://discord.comma.ai" target="_blank">Discord server</a> and go to
        the <strong>#hw-three-3x</strong> channel.
      </p>
    </section>

    <div class="hidden lg:block">
      <hr />
      flash.comma.ai version: <code>{version}</code>
    </div>
  </main>

  <div class="lg:flex-1 h-[700px] lg:h-screen bg-gray-100 dark:bg-gray-800">
    {#if loading}
      <p className="text-black dark:text-white">Loading...</p>
    {:else if error}
      <p className="text-black dark:text-white">Error loading flash</p>
    {:else}
      <svelte:component this={FlashComponent} />
    {/if}
  </div>

  <div class="w-screen max-w-none p-12 md:p-16 prose dark:prose-invert bg-white dark:bg-gray-900 lg:hidden">
    flash.comma.ai version: <code>{version}</code>
  </div>
</div>