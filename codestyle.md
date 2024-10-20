# codestyle

## General criterion
- Abide by [Vue.js Style Guide](https://vuejs.org/v2/style-guide/)。
- Indent with **4 Spaces** (no tabs).
- Keep no more than **80 characters per line** to improve readability.

## template
- Nested elements are indent **4 Spaces**.
- Use meaningful class names to describe the purpose.
- The previous line of the different modules is commented
- Leave blank lines between different template blocks (e.g. buttons, tables) to enhance readability.

## JS
- Most functions do not use arrow function definitions
- Variable names use the hump nomenclature
- Constants use all lowercase nomenclature
- Handle asynchronous operations with async/await.
- Always use try/catch to handle potential errors

### html example
```html
    `<el-button 
        plain 
        @click="dialogFormVisible = true" 
        type="primary" 
        class="add-contact-btn">
    </el-button>`
```

### js example
```js
    `const fetchData = async () => {
        try {
            const response = await fetch(url);
            const data = await response.json();
        } catch (error) {
            console.error('get data error:', error);
        }
    };`
``` 