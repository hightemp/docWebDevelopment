https://codepen.io/micjamking/pen/QdojLz

```html
<div class="container">
	<div class="box box-1">box 1</div>
	<div class="box box-2">
		This box with a border should fill the blue area except for the padding (just to show the middle flexbox item).
	</div>
	<div class="box box-3">box 3</div>
</div>
```

```css
html,
body {
	height: 100%;
}

.container {
	height: 100%;
  min-height: 100%;
	display: flex;
	flex-direction: column;
	
	.box {
		text-align: center;
		color: white;
		font-family: sans-serif;
		font-size: 36px;
		padding: 20px;
		
		display: flex;
		flex-direction: column;
    /*justify-content: center;*/
	}
	
	.box-1 {
		background-color: green;
		height: 60px;
	}
	
	.box-2 {
		background-color: blue;
		flex: 1;
		overflow: scroll;
	}
	
	.box-3 {
		background-color: red;
		height: 60px;
	}
}
```

**********
[CSS](/tags/CSS.md)
[flex](/tags/flex.md)
