<!DOCTYPE html>
<html lang="kr">
<head>
<meta charset="utf-8"/>
<title>JavaScript Load Image</title>
<script src="js/jquery-3.5.1.min.js"></script>
<script src="js/load-image.all.min.js"></script>
<script src="js/html2canvas.js"></script>
<style>
      .box{
          height: 500px;
          width: 500px;
          border-width: thick;
          border-style: dashed;
          border-color: rgb(144, 211, 144);
          border-radius: 10px;
          text-align: center;
      }
</style>
</head>
<body>
<hr>
<form action="export.php" method="POST" enctype="multipart/form-data">
	<input type="file" id="file-input" name="file">
	<input type="button" id="button" value="회전">
	<input type="submit" id="saveimg" value="업로드">
  <button id = "down" type="button">다운로드</button>
	<br>
	회전각도 : <input type="text" id="rotatetophp" name="rtp" value=0>
	원본EXIF : <input type="text" id="ori" name="ori" value=0>
	<hr>

	<div class="box"> <!--사진 틀 div-->
		<img id="thumb" > <!--사진이 들어갈 img 태그-->
	</div>
  
	<hr>
</form>

<script>



$('#file-input').on('change', function fileChangeHandler(event) {
    event.preventDefault()
    var originalEvent = event.originalEvent
    var target = originalEvent.dataTransfer || originalEvent.target
    var file = target && target.files && target.files[0]
    if (!file) {
      return
    }
    displayImage(file)
  }
)

//객체화된 파일을 그릴 옵션 생성
function displayImage(file) {
  var options = {
    maxWidth: 500, maxHeight : 500, //크기 조절
    canvas: true,
    orientation: Number(0) || true, //orientation 값 보정
    meta: true
  }//updateResults 함수 실행
  if (!loadImage(file, updateResults, options)) {
    removeMetaData()
  }
}

function updateResults(img, data, keepMetaData) {
  //원본 Orientation 값 읽어오기
  if (!keepMetaData) {
    if(data){
      if (!data) return
      var exif = data.exif
      if (exif){//읽어온 원본 Orientation 값을 input text 태그(ori)에 삽입
        var ori_val = parseInt(exif.get('Orientation'));
          if(ori_val == 0 || ori_val ==1) ori_val = "회전값 없음";
          else if(ori_val == 2) ori_val = "2 (좌우반전)";
          else if(ori_val == 3) ori_val = "3 (180º)";
          else if(ori_val == 4) ori_val = "4 (180º&좌우반전)";
          else if(ori_val == 5) ori_val = "5 (270º&좌우반전)";
          else if(ori_val == 6) ori_val = "6 (270º)";
          else if(ori_val == 7) ori_val = "7 (90º&좌우반전)";
          else if(ori_val == 8) ori_val = "8 (90º)";
          document.getElementById("ori").value = ori_val;
      }
    }
  }
  if (data.imageHead&&data.exif) {
      //브라우저 자체 보정을 피하기 위해 Orientation값 초기화
      loadImage.writeExifData(data.imageHead, data, 'Orientation', 1)
    }
  //img 태그에 보정된 이미지 적용
  document.querySelector("#thumb").src = img.toDataURL("image/jpeg");
}

$('#button').on('click',function rotate(){
  //rotatetophp = php에 넘겨줄 회전 값 (버튼 누를때마다 증가)
  var num = document.getElementById("rotatetophp").value;
  document.getElementById("rotatetophp").value = (parseInt(num)+90) % 360;
  var img = $('#thumb')[0]; // 보정된 img 태그(thumb)객체화
  if (img) {
    updateResults(
      loadImage.scale(img, {
        maxWidth: 500 ,maxheight: 500 , //크기 조절
        orientation: Number(8) || true, //orientation 8 = 좌측 90도
      }),
      true
    )
  }
})

//이미지(png)로 다운로드
$(function(){
$("#down").click(function() { 
  html2canvas($("#thumb")[0]).then(function(canvas){
          var myImage = canvas.toDataURL();
          downloadURI(myImage, "다운로드.png") 
      });
 });
});

function downloadURI(uri, name){
      var link = document.createElement("a")
      link.download = name;
      link.href = uri;
      document.body.appendChild(link);
      link.click();
  }

</script>
</body>
</html>
