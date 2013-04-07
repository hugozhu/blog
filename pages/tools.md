---
title : Tools
description: Very helpful tools for development
---

{{>right_ads}}

# URL Encode/Decode

<script type="text/javascript">
function encode() {
    var obj = document.getElementById('dencoder');
    var unencoded = obj.value;
    obj.value = encodeURIComponent(unencoded);
}
function decode() {
    var obj = document.getElementById('dencoder');
    var encoded = obj.value;
    obj.value = decodeURIComponent(encoded.replace(/\+/g,  " "));
}
</script>

<form onsubmit="return false;">
<textarea cols="200" rows="5" id="dencoder" style="width:500px;height:50px"></textarea>    
<div>
<input type="button" onclick="decode()" value="Decode">
<input type="button" onclick="encode()" value="Encode">
</div>
</form>

# JSON Pretty Print

