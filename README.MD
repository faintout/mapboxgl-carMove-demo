使用mapboxgl实现实时车辆移动效果，并优化卡顿

先来看下demo效果：

<p align="center">
    <img src="./images/动画.gif" />
</p>

**说下思路：**


 - **数据源：**
   - 数据从websocket传递,数据为多个车辆的数组形式。
 - **添加及更新**：
   - 数据过来判断一下 在地图上是否有当前车辆Id的车
      - 如果有则更新当前车的位置。
      - 如果没有则把当前车辆添加到地图上

 - **清除**：
   - 过滤地图上车辆的列表，找到更新数据的列表中不包含的id，这部分就是在后续的更新中不在地图上的。
   - 清除该部分在地图上的车

**直接上代码**
```javascript

        //定义基础数据
        //存放marker数据
        const markers = {}
        let dataIndex = 0
        let carMapDataList = []

        //创建marker
        // coordinates Array<Number,Number>
        const createMarker = (coordinates) => {
            const el = document.createElement('div');
            el.className = 'marker';
            el.style.backgroundImage = 'url(./images/car.png)';
            el.style.width = '20px';
            el.style.height = '10px';
            el.style['background-size'] = 'cover'
            el.style['color'] = '#ffffff'
            return new mapboxgl.Marker(el).setLngLat(coordinates)
        }
        /**
         * @description 更新车辆位置
         * @param {Number} vehicleId  车辆的id
         * @param {Array<Number,Number>} newCoordinates  车辆位置数据[lng,lat]
         * @param {Number} rotation  车辆的旋转角度
         * @return {void} 
        */
        const updateVehiclePosition = (vehicleId, newCoordinates, rotation = 0) => {
            // 如果车辆图标已经存在，则直接更新位置
            if (markers.hasOwnProperty(vehicleId)) {
                markers[vehicleId].setLngLat(newCoordinates).setRotation(rotation);
            } else {
                // 否则创建新的车辆图标并添加到地图上
                markers[vehicleId] = createMarker(newCoordinates).addTo(map);
            }
        }

        /**
         * @description 清除地图中不在 WebSocket 列表中的车辆图标
         * @param {Array} usedVehicleIds  更新数据时列表ID的集合
         * @return {void} 
        */
        const clearUnusedMarkers = (usedVehicleIds) => {
            for (const vehicleId in markers) {
                if (!usedVehicleIds.includes(vehicleId - 0)) {
                    markers[vehicleId].remove();
                    delete markers[vehicleId];
                }
            }
        }

        //处理所需数据
        // dataList Array
        const makeVehiclePosition = (dataList) => {
            //清除所有地图车辆
            if (dataList === undefined) {
                clearUnusedMarkers([]);
                return
            }
            // 存储 WebSocket 接收到的车辆ID
            const usedVehicleIds = [];
            dataList.map(data => {
                const { id, longitude, latitude, angle } = data
                updateVehiclePosition(id, [longitude, latitude], angle);
                // 将车辆ID添加到使用的车辆ID列表中
                usedVehicleIds.push(id);
            })
            // 清除不在 WebSocket 列表中的车辆图标
            clearUnusedMarkers(usedVehicleIds);
        }
        //获取车辆数据
        const getCarDataList = ()=>{
            fetch('./json/data.json')
            .then(response => response.json())
            .then(data => {
                // 在这里可以使用获取到的JSON数据
                carMapDataList = data
                requestAnimationFrame(cycleUpdateData)
            })
            .catch(error => {
                // 处理错误
                console.error('发生错误:', error);
            });
        }
        //循环更新数据
        const cycleUpdateData = ()=>{
            makeVehiclePosition(carMapDataList[dataIndex])
            dataIndex = (dataIndex+1)%carMapDataList.length
            requestAnimationFrame(cycleUpdateData)
        }

```

**对于数据过多造成卡顿的问题：**

这个在最初实现功能的时候出现过，最初使用source源和layer的方式添加及setData更新的方式，但数据过多时页面卡顿严重，后续经过几个方法的实现，最后上文中是最优方式，目前demo中数据量也不会造成卡顿。

**最后附上demo地址：**

**以服务形式启动就可以查看了**

**https://github.com/faintout/mapboxgl-carMove-demo**




