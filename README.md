# 3D 렌더링 프로그램

![preview](https://user-images.githubusercontent.com/96270683/188530821-2ff262f7-b663-4068-b590-61707b66adc6.png)

## 소개
OpenGL 라이브러리를 통해 직접 구현한 3D 렌더링 프로그램입니다.


프로그램을 실행하시려면 3DRenderingProgram.zip을 다운받아 압축을 푸신 후, 폴더 내 exe파일을 실행하시면 됩니다.

## 기능
:heavy_check_mark: 삼각형 그리기


:heavy_check_mark: 이동, 회전, 스케일 변환


:heavy_check_mark: 카메라 투영


:heavy_check_mark: 텍스쳐 매핑


:heavy_check_mark: 라이팅


:heavy_check_mark: 모델 로딩


:heavy_check_mark: 셰도우 맵


:heavy_check_mark: Skybox


## 상세 설명
### 삼각형 그리기
삼각형의 버텍스 데이터를 버퍼에 입력하고 Fragment Shader에 컬러값을 빨간색으로 설정합니다.


![opengl_triangle](https://user-images.githubusercontent.com/96270683/188780416-24783747-a690-4d49-8583-257063ae0eb6.PNG)
``` c++
void CreateTriangle()
{
	GLfloat vertices[] = {
		-1.0f, -1.0f, 0.0f,
		1.0f, -1.0f, 0.0f,
		0.0f, 1.0f, 0.0f
	};

	glGenVertexArrays(1, &VAO);
	glBindVertexArray(VAO);

	glGenBuffers(1, &VBO);
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(0);

	glBindBuffer(GL_ARRAY_BUFFER, 0);

	glBindVertexArray(0);
}
```
### 이동, 회전, 스케일 변환
이동, 회전, 스케일 변환행렬을 적용하여 삼각형의 위치, 각도, 비율을 조정합니다.


![opengl_scale](https://user-images.githubusercontent.com/96270683/188781465-f34c0fa9-517d-4b47-96a1-eeaa33212a2a.PNG)
``` c++
glm::mat4 model(1.0f);

model = glm::rotate(model, curAngle * toRadians, glm::vec3(0.0f, 0.0f, 1.0f));
model = glm::translate(model, glm::vec3(triOffset, 0.0f, 0.0f));
model = glm::scale(model, glm::vec3(curSize, 0.4f, 0.0f));

glUniformMatrix4fv(uniformModel, 1, GL_FALSE, glm::value_ptr(model));
```
		
### 카메라 투영
카메라를 perspective로 투영합니다.



![opengl_projection](https://user-images.githubusercontent.com/96270683/188787274-6c6570e3-9cf4-43d0-acc4-535d8760e7ab.PNG)
``` c++
glm::mat4 projection = glm::perspective(glm::radians(45.0f), (GLfloat)bufferWidth / (GLfloat)bufferHeight, 0.1f, 100.0f);
glUniformMatrix4fv(uniformProjection, 1, GL_FALSE, glm::value_ptr(projection));
```
### 텍스쳐 매핑
### 라이팅
### 모델 로딩
### 셰도우 맵
### Skybox

