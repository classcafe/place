<?php

	//템플릿(전체 페이지)

	include_once $_SERVER['DOCUMENT_ROOT'] . "/inc/init.php";
	include_once $_SERVER['DOCUMENT_ROOT'] . "/inc/DAO.class.php";
	include_once $_SERVER['DOCUMENT_ROOT'] . "/inc/MarkDownDAO.class.php";

?>
<!doctype html>
<html lang="en">
<head>
<?php include_once $_SERVER['DOCUMENT_ROOT'] . "/inc/header.php" ?>
<style>

</style>
<script type="text/javascript" src="//dapi.kakao.com/v2/maps/sdk.js?appkey=f370e3b9ea581ebebf94f6adb5058a94&libraries=services"></script>
<script>
	//아래의 마커 리스트
	var markerList = [];
</script>

</head>
<body oncontextmenu='return false' ondragstart='return false' onselectstart='return false'>

<?php include_once $_SERVER['DOCUMENT_ROOT'] . "/inc/nav.php" ?>

<section id="main">
	<div id="left">
		<i class="fas fa-map-marker-alt" id="lefticon"></i>
	</div>
	<div id="right">
		<div id="map" style="width: 1000px; height:400px; border: 1px solid #686E80; border-radius: 2px;"></div>
		
		<button type="button" class="btn btn-secondary" data-toggle="modal" data-target="#exampleModal" style="position: absolute; left: 1376px;top: 75px; opacity: .74;">
			<i class="fas fa-map-marker-alt"></i>
		</button>

		<div>
		<?php

			//특정 폴더(문제별 .md) 목록 읽기
			$filelist = $pinn->filelist('/place');
			//asort($filelist);
			sort($filelist);

			foreach($filelist as $file) {
				if ($pinn->endsWith($file, ".md")) {

					//내용 읽기
					$fdao = new DAO("/place/" . $file, 'r');
					$ftemp = explode("|", $fdao->read());
					
					//요소 시작
					echo "<div onclick='moveToMarker" . $ftemp[2] . ";' class='placeItem'>";

					//타입 구분
					if ($ftemp[1] == "1") {
						echo "<i class='fas fa-utensils'></i> ";
					} else {
						echo "<i class='fas fa-coffee'></i> ";
					}

					//요소 내용(상호명)
					echo "<b>" . $ftemp[0] . "</b>" . " " . str_replace("\r\n", " | ", $ftemp[3]);
					
					//요소 끝
					echo "</div>";



					//마커 추가
					echo "<script>";
					echo "var templatlng = new daum.maps.LatLng" . $ftemp[2] . ";";
					echo "var tempMarker = new daum.maps.Marker({ position: templatlng });";
					echo "tempMarker.setTitle('" . $ftemp[0] . "');";
					echo "var tempImg = new daum.maps.MarkerImage('/images/marker" . $ftemp[1] . ".png', new daum.maps.Size(30, 38),  new daum.maps.Point(15, 38));";
					echo "tempMarker.setImage(tempImg);";
					echo "markerList.push(tempMarker);";
					echo "</script>";

				}
			}
			?>

			<style>

				.placeItem {
					margin: 5px;
					font-size: 14px;
					cursor: pointer;
					opacity: .74;
				}

				.placeItem:hover {
					opacity: 1;
				}

				.placeItem:first-child {
					margin-top: 25px;
				}

				.placeItem i {
					display: inline-block;
					text-align: center;
					width: 20px;
				}
			</style>
			<script>
					
			</script>
		</div>

		<!-- Modal -->
		<div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
			<div class="modal-dialog modal-lg" role="document" style="width: 800px;margin-top: 50px;">
				<div class="modal-content" style="width: 800px;">
					<div class="modal-header" style="background-color: #686E80; padding: 5px 20px;">
						<h5 class="modal-title" id="exampleModalLabel" style="color: white;"><i class="fas fa-map-marker-alt"></i> Add Place</h5>
						<button type="button" class="close" data-dismiss="modal" aria-label="Close"style="color: white;">
							<span aria-hidden="true">&times;</span>
						</button>
					</div>
				<div class="modal-body">
					<form method="post" action="placeok.php" id="placeform">
						<div id="map2" style="width:740px; height: 300px; margin: 10px auto;"></div>
						<hr>
						<div id="selcategory" class="btn-group" role="group" aria-label="Basic example">
							
							<input type="text" class="form-control" style="margin-right: 5px; width: 400px;" placeholder="Place Name" id="placeName" name="placeName">
							<button type="button" class="btn btn-secondary" style="opacity:1; border-radius:0;" data-type="1" onclick="setType(1);"><i class="fas fa-utensils"></i></button>
							<button type="button" class="btn btn-secondary" style="opacity:.5; border-radius:0;" data-type="2" onclick="setType(2);"><i class="fas fa-coffee"></i></button>
							<input type="hidden" name="placeType" id="placeType" value="1">
							<input type="hidden" name="placePoint" id="placePoint">
							
						</div>
						<textarea class="form-control" style="margin: 10px 0px; height: 100px;" placeholder="Place Description" id="placeDescription" name="placeDescription"></textarea>
					</form>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
					<button type="button" class="btn btn-danger" style="opacity: .74;" onclick="savePoint();">Save Place</button>
				</div>
				</div>
			</div>
		</div>

		
		<script>

			$("#selcategory > button").click(function() {
				$("#selcategory > button").css("opacity", "0.5");
				$(this).css("opacity", "1");
				mimg = $(this).data("type");
				changeMarker();
			});
			

			var container = document.getElementById('map');
			var options = {
			center: new daum.maps.LatLng(37.499297, 127.033203),
				level: 3
			};
			var map = new daum.maps.Map(container, options);


			
			var map2;
			var tempMarker;
			var mimg = 1;
			var marker;

			$('#exampleModal').on('shown.bs.modal', function () {


				var container2 = document.getElementById("map2");
				
				var options2 = {
					center: new daum.maps.LatLng(37.499297, 127.033203),
					level: 3
				};

				map2 = new daum.maps.Map(container2, options2);
				
				//클릭 이벤트 + 마커 추가하기
				daum.maps.event.addListener(map2, "click", function(evt) {
					
					//이전 마커 삭제하기
					if (tempMarker != null) {
						tempMarker.setMap(null); //삭제
					}
					
					//새 마커 추가하기
					//if (!flag) {
						var latlng = new daum.maps.LatLng(evt.latLng.getLat()
															, evt.latLng.getLng());
						marker = new daum.maps.Marker({ position: latlng });
						$("#placePoint").val(latlng);

						var markerImage = new daum.maps.MarkerImage(
																	"/images/marker" + mimg + ".png",
																	new daum.maps.Size(30, 38), 
																	new daum.maps.Point(15, 38));
						marker.setImage(markerImage);
						
						marker.setDraggable(true);

						daum.maps.event.addListener(marker, 'dragend', function() {
							$("#placePoint").val(marker.getPosition());
						});


						marker.setMap(map2);
						
						tempMarker = marker; //새 마커를 임시 변수 복사
						//flag = true;
						
						
					//}
					
				});
				
			});
			
			function changeMarker() {
				var markerImage = new daum.maps.MarkerImage(
																	"/img/marker" + mimg + ".png",
																	new daum.maps.Size(30, 38), 
																	new daum.maps.Point(15, 38));
				marker.setImage(markerImage);
			}



			//장소 추가하기
			function savePoint() {
				//alert($("#placeName").val());
				//alert($("#placeType").val());
				//alert($("#placeDescription").val());
				//alert($("#placePoint").val());
				$("#placeform").submit();
			}

			//장소 타입 선택하기
			function setType(no) {
				$("#placeType").val(no);
			}



			//불러온 마커 배열 출력하기
			markerList.forEach(function (item, index) {
				//console.log(item);
				//console.log(item.constructor);
				item.setMap(map);
			});


			//클릭한 마커 위치로 이동하기
			function moveToMarker(lat, lng) {
				
				$('html, body').animate({
					scrollTop: 0
				}, 100, function() { 
					var moveLatLng = new daum.maps.LatLng(lat, lng);   
					map.panTo(moveLatLng);
				});

			}

		</script>
	</div>
</section>

<script>
	$("#main, #left, #right").height($(document).height() + 50);
</script>
</body>
</html>
