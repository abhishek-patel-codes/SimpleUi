
### Questions? 

<details>
<summary>dev-installation (click to expand)</summary>

```shell
pip uninstall -y selenium-driverless
```
</details>

## Sponsors

This project is currently not being sponsored.

### Dependencies

* [Python >= 3.8](https://www.python.org/downloads/)

### Installing

* Install [Google-Chrome](https://www.google.de/chrome/)
* ```pip install selenium-driverless```


### Usage

### with asyncio
```python
from selenium_driverless import webdriver
from selenium_driverless.types.by import By
import asyncio


async def main():
    options = webdriver.ChromeOptions()
    async with webdriver.Chrome(options=options) as driver:
        await driver.get('http://nowsecure.nl#relax', wait_load=True)
        await driver.sleep(0.5)
        await driver.wait_for_cdp("Page.domContentEventFired", timeout=15)
        
        # wait 10s for elem to exist
        elem = await driver.find_element(By.XPATH, '/html/body/div[2]/div/main/p[2]/a', timeout=10)
        await elem.click(move_to=True)

        alert = await driver.switch_to.alert
        print(alert.text)
        await alert.accept()

        print(await driver.title)


asyncio.run(main())

```

### synchronous
asyncified, bugs are to expect

<details>

<summary>example code</summary>

```python
from selenium_driverless.sync import webdriver

options = webdriver.ChromeOptions()
with webdriver.Chrome(options=options) as driver:
    driver.get('http://nowsecure.nl#relax')
    driver.sleep(0.5)
    driver.wait_for_cdp("Page.domContentEventFired", timeout=15)

    title = driver.title
    url = driver.current_url
    source = driver.page_source
    print(title)
```

</details>

### custom debugger address
```python
from selenium_driverless import webdriver

options = webdriver.ChromeOptions()
options.debugger_address = "127.0.0.1:2005"

# specify if you don't want to run remote
# options.add_argument("--remote-debugging-port=2005")

async with webdriver.Chrome(options=options) as driver:
  await driver.get('http://nowsecure.nl#relax', wait_load=True)
```

## Multiple tabs simultaneously
Note: asyncio is recommended, threading only works on independent webdriver.Chrome instances.

<details>
<summary>Example Code (Click to expand)</summary>

```python
from selenium_driverless.sync import webdriver
from selenium_driverless.utils.utils import read
from selenium_driverless import webdriver
import asyncio


async def target_1_handler(target):
    await target.get('https://abrahamjuliot.github.io/creepjs/')
    print(await target.title)


async def target_2_handler(target):
    await target.get("about:blank")
    await target.execute_script(await script=read("/files/js/show_mousemove.js"))
    await target.pointer.move_to(500, 500, total_time=2)


async def main():
    options = webdriver.ChromeOptions()
    async with webdriver.Chrome(options=options) as driver:
        target_1 = await driver.current_target
        target_2 = await driver.new_window("tab", activate=False)
        await asyncio.gather(
            target_1_handler(target_1),
            target_2_handler(target_2)
        )
        await target_1.focus()
        input("press ENTER to exit")


asyncio.run(main())
```

</details>

### Isolated execution contexts
- execute `javascript` without getting detected ( in a **isolated world**)
<details>
<summary>Example Code (Click to expand)</summary>

```python
from selenium_driverless.sync import webdriver
from selenium_driverless import webdriver
import asyncio


async def main():
    options = webdriver.ChromeOptions()
    async with webdriver.Chrome(options=options) as driver:
        await driver.get('chrome://version')
        script = """
        const proxy = new Proxy(document.documentElement, {
          get(target, prop, receiver) {
            if(prop === "outerHTML"){
                console.log('detected access on "'+prop+'"', receiver)
                return "mocked value:)"
            }
            else{return Reflect.get(...arguments)}
          },
        });
        Object.defineProperty(document, "documentElement", {
          value: proxy
        })
        """
        await driver.execute_script(script)
        src = await driver.execute_script("return document.documentElement.outerHTML", unique_context=True)
        mocked = await driver.execute_script("return document.documentElement.outerHTML", unique_context=False)
        print(src, mocked)


asyncio.run(main())
```

</details>

### Pointer Interaction

```python
pointer = driver.current_pointer
move_kwargs = {"total_time": 0.7, "accel": 2, "smooth_soft": 20}

await pointer.move_to(100, 500)
await pointer.click(500, 50, move_kwargs=move_kwargs, move_to=True)
```
### Iframes / Frames
due `swtich_to.frame()` being deprecated for driverless, use this instead

```python
iframes = await driver.find_elements(By.TAG_NAME, "iframe")
await asyncio.sleep(0.5)
iframe_document = await iframes[0].content_document
# iframe_document.find_elements(...)
```

### use preferences
```python
from selenium_driverless import webdriver
options = webdriver.ChromeOptions()

 # recommended usage
options.update_pref("download.prompt_for_download", False)
# or
options.prefs.update({"download": {"prompt_for_download": False}})

# supported
options.add_experimental_option("prefs", {"download.prompt_for_download": False})
```

### Multiple Contexts
- different cookies for each context
- A context can have multiple windows and tabs within
- different proxy for each context
- opens as a window as incognito
<details>
<summary>Example Code (Click to expand)</summary>

```python
from selenium_driverless import webdriver
import asyncio




asyncio.run(main())
```
</details>

#### Custom exception handling
You can implement custom exception handling as following

```python
import selenium_driverless
import sys
handler = (lambda e: print(f'Exception in event-handler:\n{e.__class__.__module__}.{e.__class__.__name__}: {e}',
                           file=sys.stderr))
sys.modules["selenium_driverless"].EXC_HANDLER = handler
sys.modules["cdp_socket"].EXC_HANDLER = handler