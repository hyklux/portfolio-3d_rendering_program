# 3D Rendering Program

![opengl_final](https://user-images.githubusercontent.com/96270683/188821931-f96d2f21-9546-48a2-9651-fef8c7cb5d18.PNG)
## Introduction
This is a 3D rendering program directly implemented through the OpenGL library.


To run the program, download 3DRenderingProgram.zip, unzip it, and run the exe file in the folder.

## Implmentations
:heavy_check_mark: Drawing a triangle


:heavy_check_mark: Translation, rotation, scale transformation


:heavy_check_mark: Camera projection


:heavy_check_mark: Texture mapping


:heavy_check_mark: Lighting


:heavy_check_mark: Loading models


:heavy_check_mark: Shadow Map


:heavy_check_mark: Skybox


## Drawing a triangle
Put the vertex data of the triangle into the buffer and set the color value to red in the fragment shader.


![opengl_triangle](https://user-images.githubusercontent.com/96270683/188780416-24783747-a690-4d49-8583-257063ae0eb6.PNG)
``` c++
void CreateTriangle()
{
	GLfloat vertices[] = {
		-1.0f, -1.0f, 0.0f,
		1.0f, -1.0f, 0.0f,
		0.0f, 1.0f, 0.0f
	};

	//Create VAOs
	glGenVertexArrays(1, &VAO);
	//Connect the VAO we created to the current editable
	glBindVertexArray(VAO);

	//Create VBOs
	glGenBuffers(1, &VBO);
	//Connect the VBO we created to the current editable
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	
	//Store triangle vertex info in VBO
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
	
	//Attribute 0 tells the VAO to separate the elements of vertices by 3 and interpret them as vertices' positions.
	//(VAO 색인 값, 좌표수(x, y, z), 데이터 타입, 정상화 여부, 바이트 오프셋, 시작 색인 값
	//(VAO index value, number of coordinates (x, y, z), data type, normalization, byte offset, start index value)
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
	//Allow use of VAO
	glEnableVertexAttribArray(0);

	//End VBO modification and initialize connection
	glBindBuffer(GL_ARRAY_BUFFER, 0);
	//End and connect VAO fix
	glBindVertexArray(0);
}

void Render()
{
    	//Reset background color
    	glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    	//Initialize color buffer and depth buffer
    	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    	//Data connection in VBO
    	glBindVertexArray(VBO);
    	//Draw according to the vertex data
    	glDrawArrays(GL_TRIANGLES, 0, 3);
    	//Disconnect vertex data
    	glBindVertexArray(0);
}
```
## Translation, rotation, scale transformation
Adjust the position, angle, and scale of triangles by applying translation, rotation, and scale transformation matrices.


![opengl_scale](https://user-images.githubusercontent.com/96270683/188781465-f34c0fa9-517d-4b47-96a1-eeaa33212a2a.PNG)
``` c++
glm::mat4 model(1.0f);

//Rotation
model = glm::rotate(model, curAngle * toRadians, glm::vec3(0.0f, 0.0f, 1.0f));
//Translation
model = glm::translate(model, glm::vec3(triOffset, 0.0f, 0.0f));
//Scale transformation
model = glm::scale(model, glm::vec3(curSize, 0.4f, 0.0f));

//Pass uniformModel value to vertex shader
glUniformMatrix4fv(uniformModel, 1, GL_FALSE, glm::value_ptr(model));
```
		
## 카메라 투영
카메라를 원근(perspective)으로 투영합니다. 투영 행렬 * 뷰 스페이스 행렬 * 월드 스페이스 행렬을 연산하여 vertex의 최종 위치를 계산합니다.  



![opengl_projection](https://user-images.githubusercontent.com/96270683/188787274-6c6570e3-9cf4-43d0-acc4-535d8760e7ab.PNG)
``` c++
model = glm::mat4(1.0f);
model = glm::translate(model, glm::vec3(0.0f, 1.0f, -2.5f));
model = glm::scale(model, glm::vec3(0.4f, 0.4f, 1.0f));

//카메라 설정
camera = Camera(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f), -90.0f, 0.0f, 5.0f, 0.5f);

//투영 정보 설정(fov: 60도, 종횡비: 화면 가로/세로, Near Plane: 0.1f, Far Plane: 100.0 
glm::mat4 projection = glm::perspective(glm::radians(60.0f), (GLfloat)bufferWidth / (GLfloat)bufferHeight, 0.1f, 100.0f);

//vertex shader에 model(월드 스페이스), projection(투영), view(뷰 스페이스) 값을 넘겨주기
glUniformMatrix4fv(uniformModel, 1, GL_FALSE, glm::value_ptr(model));
glUniformMatrix4fv(uniformProjection, 1, GL_FALSE, glm::value_ptr(projection));
glUniformMatrix4fv(uniformView, 1, GL_FALSE, glm::value_ptr(camera.calculateViewMatrix()));
```
- Vertex Shader
``` c++
#version 330

layout (location = 0) in vec3 pos;

out vec4 vCol;

uniform mat4 model;
uniform mat4 projection;
uniform mat4 view;

void main()
{
	//투영 행렬(projection) * 뷰 스페이스 행렬(view) * 월드 스페이스 행렬(model)을 연산하여 vertex의 최종 위치를 계산
	gl_Position = projection * view * model * vec4(pos, 1.0);
	vCol = vec4(clamp(pos, 0.0f, 1.0f), 1.0f);
}
```
## 텍스쳐 매핑
텍스처 파일을 로드하여 모델에 적용합니다.


![opengl_texture](https://user-images.githubusercontent.com/96270683/188788484-c7ed729c-45e8-4922-a0e0-be2ce415ed27.PNG)
- Texture.cpp
``` c++
void Texture::LoadTexture()
{
	unsigned char *texData = stbi_load(fileLocation, &width, &height, &bitDepth, 0);
	if (!texData)
	{
		printf("Failed to find: %s\n", fileLocation);
		return;
	}

	glGenTextures(1, &textureID);
	glBindTexture(GL_TEXTURE_2D, textureID);

	/**텍스쳐 매핑 옵션 설정!*/
	//가로 범위가 [0.0,1.0]을 벗어나게 되면 GL_REPEAT 옵션 적용
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
	//세로 범위가 [0.0,1.0]을 벗어나게 되면 GL_REPEAT 옵션 적용
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
	//폴리곤이 텍스처보다 작을 때 GL_LINEAR 옵션 적용
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	//폴리곤이 텍스처보다 클 때 GL_LINEAR 옵션으로 적용
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

	//로드한 텍스쳐에 대한 렌더링 옵션 설정
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, texData);
	
	//mipmap 생성
	glGenerateMipmap(GL_TEXTURE_2D);

	glBindTexture(GL_TEXTURE_2D, 0);
	stbi_image_free(texData);
}

void Texture::UseTexture()
{
	glActiveTexture(GL_TEXTURE0);
	glBindTexture(GL_TEXTURE_2D, textureID);
}
```
- Vertex Shader
``` c++
#version 330

layout (location = 0) in vec3 pos;
layout (location = 1) in vec2 tex;

out vec4 vCol;
out vec2 TexCoord;

uniform mat4 model;
uniform mat4 projection;
uniform mat4 view;

void main()
{
	gl_Position = projection * view * model * vec4(pos, 1.0);
	vCol = vec4(clamp(pos, 0.0f, 1.0f), 1.0f);
	
	TexCoord = tex;
}
```
- Fragment Shader
``` c++
#version 330

in vec4 vCol;
in vec2 TexCoord;

out vec4 colour;

uniform sampler2D theTexture;

void main()
{
	colour = texture(theTexture, TexCoord);
}
```
## 라이팅
라이팅 구현에는 Phong Lighting Model(Ambient + Diffuse + Specular)을 사용합니다.
### Ambient


![lighting_ambient](https://user-images.githubusercontent.com/96270683/235663786-f9c6df61-591b-4d50-ab7c-0953911bbcab.png)
### Diffuse


![lighting_diffuse](https://user-images.githubusercontent.com/96270683/235663845-1bb6a82f-8359-4038-befd-6d80e18e1d6e.png)
### Specular


![lighting_speccular](https://user-images.githubusercontent.com/96270683/235663875-c334c1d8-7823-4ee2-8be7-8bb7490ca465.png)
### Directional Light
Fragment Shader에서 Ambient, Diffuse, Specular 라이트 연산을 각각 처리하여 Directional Light를 구현합니다.


![opengl_directional_light](https://user-images.githubusercontent.com/96270683/188805453-cd1a67df-6daf-400a-8efc-c726738456e1.PNG)
- Fragment Shader
``` c++
void main()
{
	/**Ambient Light 연산 (빛의 색 * ambient 강도)*/
	vec4 ambientColour = vec4(directionalLight.colour, 1.0f) * directionalLight.ambientIntensity;
	
	/**Diffuse Light 연산 (빛의 색 * diffuse 강도 * diffuse 비율)*/
	float diffuseFactor = max(dot(normalize(Normal), normalize(directionalLight.direction)), 0.0f); //법선 벡터와 빛 방향  내적을 통해 diffuseFactor 계산
	vec4 diffuseColour = vec4(directionalLight.colour, 1.0f) * directionalLight.diffuseIntensity * diffuseFactor;
	
	/**Specular Light 연산  (빛의 색 * specular 강도 * shininess를 적용한 specular 비율)*/
	vec4 specularColour = vec4(0, 0, 0, 0);
	if(diffuseFactor > 0.0f)
	{
		vec3 fragToEye = normalize(eyePosition - FragPos);
		vec3 reflectedVertex = normalize(reflect(directionalLight.direction, normalize(Normal)));
		
		//fragment에서 카메라를 향하는 벡터와 빛의 반사 벡터의 내적을 통해 specularFactor 계산
		float specularFactor = dot(fragToEye, reflectedVertex);
		if(specularFactor > 0.0f)
		{
			specularFactor = pow(specularFactor, material.shininess);
			specularColour = vec4(directionalLight.colour * material.specularIntensity * specularFactor, 1.0f);
		}
	}
	
	colour = texture(theTexture, TexCoord) * (ambientColour + diffuseColour + specularColour);
}
```
### Point Light
Point Light는 Directional Light 연산에 빛 감쇠(Attenuation) 연산을 더하여 처리합니다.


![opengl_point_light](https://user-images.githubusercontent.com/96270683/188807781-70477324-3bf0-4606-a922-3633226c5802.PNG)
- Fragment Shader
``` c++
//Point Light 연산(Directional Light 연산 + 빛 감쇠 연산)
vec4 CalcPointLights()
{
	vec4 totalColour = vec4(0, 0, 0, 0);
	for(int i = 0; i < pointLightCount; i++)
	{
		vec3 direction = FragPos - pointLights[i].position;
		
		//Point Light로부터의 거리 계산
		float distance = length(direction);
		
		//정규화된 Point Light로부터의 방향
		direction = normalize(direction);
		
		//Directional Light 요소 연산
		vec4 colour = CalcLightByDirection(pointLights[i].base, direction);
		
		//이차방정식을 이용하여 빛 감쇠 값 유도
		float attenuation = pointLights[i].exponent * distance * distance +
							pointLights[i].linear * distance +
							pointLights[i].constant;
		
		totalColour += (colour / attenuation);
	}
	
	return totalColour;
}

//ambient, diffuse, specular 요소 연산
vec4 CalcLightByDirection(Light light, vec3 direction)
{
	//ambient 라이팅 계산
	vec4 ambientColour = vec4(light.colour, 1.0f) * light.ambientIntensity;
	
	//diffuse 라이팅 계산
	float diffuseFactor = max(dot(normalize(Normal), normalize(direction)), 0.0f);
	vec4 diffuseColour = vec4(light.colour * light.diffuseIntensity * diffuseFactor, 1.0f);
	
	//specular 라이팅 계산
	vec4 specularColour = vec4(0, 0, 0, 0);
	if(diffuseFactor > 0.0f)
	{
		vec3 fragToEye = normalize(eyePosition - FragPos);
		vec3 reflectedVertex = normalize(reflect(direction, normalize(Normal)));
		
		float specularFactor = dot(fragToEye, reflectedVertex);
		if(specularFactor > 0.0f)
		{
			specularFactor = pow(specularFactor, material.shininess);
			specularColour = vec4(light.colour * material.specularIntensity * specularFactor, 1.0f);
		}
	}

	//모든 라이팅 연산값의 합 반환
	return (ambientColour + diffuseColour + specularColour);
}

void main()
{
	//Directional Light 연산 결과 적용
	vec4 finalColour = CalcDirectionalLight();
	//Point Lights 연산 결과 적용
	finalColour += CalcPointLights();
	
	colour = texture(theTexture, TexCoord) * finalColour;
}
```
### Spot Light
Spot Light를 구현합니다.


![opengl_spot_light](https://user-images.githubusercontent.com/96270683/188808820-5160caeb-7ccd-42e2-bf3b-7566f561c2e2.PNG)
``` c++
vec4 CalcSpotLight(SpotLight sLight)
{
	vec3 rayDirection = normalize(FragPos - sLight.base.position);
	float slFactor = dot(rayDirection, sLight.direction);
	
	//Spot Light 범위 안에 있으면 조명을 처리하고 그렇지 않으면 무시한다.
	if(slFactor > sLight.edge)
	{
		vec4 colour = CalcPointLight(sLight.base);
		return colour * (1.0f - (1.0f - slFactor)*(1.0f/(1.0f - sLight.edge)));
	} 
	else
	{
		return vec4(0, 0, 0, 0);
	}
}

vec4 CalcSpotLights()
{
	vec4 totalColour = vec4(0, 0, 0, 0);
	for(int i = 0; i < spotLightCount; i++)
	{		
		totalColour += CalcSpotLight(spotLights[i]);
	}
	
	return totalColour;
}

void main()
{
	//Directional Light, Point Lights, Spot Lights를 모두 계산한 값을 더해 최종 컬러값을 구한다.
	vec4 finalColour = CalcDirectionalLight();
	finalColour += CalcPointLights();
	finalColour += CalcSpotLights();
	
	colour = texture(theTexture, TexCoord) * finalColour;
}
```
## 모델 로딩


![opengl_model](https://user-images.githubusercontent.com/96270683/188810385-fb06b19d-3115-49dc-a741-64e09035da52.PNG)
``` c++
void Model::LoadModel(const std::string & fileName)
{
	//파일로부터 모델 데이터 불러오기
	Assimp::Importer importer;
	const aiScene *scene = importer.ReadFile(fileName, aiProcess_Triangulate | aiProcess_FlipUVs | aiProcess_GenSmoothNormals | aiProcess_JoinIdenticalVertices);

	if (!scene)
	{
		printf("Model (%s) failed to load: %s", fileName, importer.GetErrorString());
		return;
	}
	
	//루트 노드부터 로딩 시작
	LoadNode(scene->mRootNode, scene);
	//머티리얼 로드
	LoadMaterials(scene);
}

void Model::LoadNode(aiNode * node, const aiScene * scene)
{
	//노드에 속하는 메시 로드
	for (size_t i = 0; i < node->mNumMeshes; i++)
	{
		LoadMesh(scene->mMeshes[node->mMeshes[i]], scene);
	}
	
	//자식 노드를 재귀의 방식으로 계속 로드
	for (size_t i = 0; i < node->mNumChildren; i++)
	{
		LoadNode(node->mChildren[i], scene);
	}
}

void Model::LoadMesh(aiMesh * mesh, const aiScene * scene)
{
	std::vector<GLfloat> vertices;
	std::vector<unsigned int> indices;

	//버텍스 정보 저장
	for (size_t i = 0; i < mesh->mNumVertices; i++)
	{
		vertices.insert(vertices.end(), { mesh->mVertices[i].x, mesh->mVertices[i].y, mesh->mVertices[i].z });
		if (mesh->mTextureCoords[0])
		{
			vertices.insert(vertices.end(), { mesh->mTextureCoords[0][i].x, mesh->mTextureCoords[0][i].y });
		}
		else 
		{
			vertices.insert(vertices.end(), { 0.0f, 0.0f });
		}
		vertices.insert(vertices.end(), { -mesh->mNormals[i].x, -mesh->mNormals[i].y, -mesh->mNormals[i].z });
	}
	
	//인덱스 정보 저장
	for (size_t i = 0; i < mesh->mNumFaces; i++)
	{
		aiFace face = mesh->mFaces[i];
		for (size_t j = 0; j < face.mNumIndices; j++)
		{
			indices.push_back(face.mIndices[j]);
		}
	}

	//메시 생성
	Mesh* newMesh = new Mesh();
	newMesh->CreateMesh(&vertices[0], &indices[0], vertices.size(), indices.size());
	meshList.push_back(newMesh);
	meshToTex.push_back(mesh->mMaterialIndex);
}

void Model::LoadMaterials(const aiScene * scene)
{
	textureList.resize(scene->mNumMaterials);
	
	for (size_t i = 0; i < scene->mNumMaterials; i++)
	{
		aiMaterial* material = scene->mMaterials[i];

		textureList[i] = nullptr;

		if (material->GetTextureCount(aiTextureType_DIFFUSE))
		{
			aiString path;
			if (material->GetTexture(aiTextureType_DIFFUSE, 0, &path) == AI_SUCCESS)
			{
				int idx = std::string(path.data).rfind("\\");
				std::string filename = std::string(path.data).substr(idx + 1);

				std::string texPath = std::string("Textures/") + filename;

				textureList[i] = new Texture(texPath.c_str());

				if (!textureList[i]->LoadTexture())
				{
					printf("Failed to load texture at: %s\n", texPath);
					delete textureList[i];
					textureList[i] = nullptr;
				}
			}
		}

		if (!textureList[i])
		{
			textureList[i] = new Texture("Textures/plain.png");
			textureList[i]->LoadTextureA();
		}
	}
}

void Model::RenderModel()
{
	for (size_t i = 0; i < meshList.size(); i++)
	{
		unsigned int materialIndex = meshToTex[i];
		
		//텍스쳐 렌더
		if (materialIndex < textureList.size() && textureList[materialIndex])
		{
			textureList[materialIndex]->UseTexture();
		}

		//메시 렌더
		meshList[i]->RenderMesh();
	}
}
```
## Shadow Map
### Directional Shadow Map
Directional Light에 대한 그림자 처리에 사용


![shadow_mapping_theory_spaces](https://user-images.githubusercontent.com/96270683/236655201-5bbc73e1-525b-49b4-9616-3a962195a6c6.png)
![opengl_directional_light](https://user-images.githubusercontent.com/96270683/188812990-fb3984b6-cf9e-4c11-860b-9ef2eaf276a2.PNG)
- ShadowMap.cpp
``` c++
bool ShadowMap::Init(unsigned int width, unsigned int height)
{
	shadowWidth = width; shadowHeight = height;

	glGenFramebuffers(1, &FBO);

	//섀도우 맵 텍스쳐 설젇
	glGenTextures(1, &shadowMap);
	glBindTexture(GL_TEXTURE_2D, shadowMap);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_DEPTH_COMPONENT, width, height, 0, GL_DEPTH_COMPONENT, GL_FLOAT, nullptr);
	
	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);
	
	float borderColor[] = { 1.0f, 1.0f, 1.0f, 1.0f };
	glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);

	//프레임버퍼에 섀도우 맵 데이터 저장
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, FBO);
	glFramebufferTexture2D(GL_DRAW_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_TEXTURE_2D, shadowMap, 0);

	glDrawBuffer(GL_NONE);
	glReadBuffer(GL_NONE);

	GLenum Status = glCheckFramebufferStatus(GL_FRAMEBUFFER);

	if (Status != GL_FRAMEBUFFER_COMPLETE)
	{
		printf("Framebuffer error: %s\n", Status);
		return false;
	}

	return true;
}

void ShadowMap::Write()
{
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, FBO);
}

void ShadowMap::Read(GLenum texUnit)
{
	glActiveTexture(texUnit);
	glBindTexture(GL_TEXTURE_2D, shadowMap);
}
```
- Directional Shadow Map Vertex Shader
``` C++
#version 330

layout (location = 0) in vec3 pos;

uniform mat4 model;
uniform mat4 directionalLightTransform; //광원의 위치로의 변환 행렬

void main()
{
	gl_Position = directionalLightTransform * model * vec4(pos, 1.0);
}
```
### Omni Directional Shadow Map
Point Light, Spot Light에 대한 그림자 처리에 사용


![point_shadows_diagram](https://user-images.githubusercontent.com/96270683/236655224-c1489793-d13c-4ebc-962f-a01d87791297.png)
![opengl_omni_shadow](https://user-images.githubusercontent.com/96270683/188812956-62a0c2da-84bc-4eba-97cb-86d2e9657d3f.PNG)
- OmniShadowMap.cpp
``` C++
bool OmniShadowMap::Init(unsigned int width, unsigned int height)
{
	shadowWidth = width; shadowHeight = height;

	glGenFramebuffers(1, &FBO);

	glGenTextures(1, &shadowMap);
	glBindTexture(GL_TEXTURE_CUBE_MAP, shadowMap);

	for (size_t i = 0; i < 6; i++)
	{
		glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_DEPTH_COMPONENT, shadowWidth, shadowHeight, 0, GL_DEPTH_COMPONENT, GL_FLOAT, nullptr);
	}

	glTexParameterf(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameterf(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);

	glBindFramebuffer(GL_FRAMEBUFFER, FBO);
	glFramebufferTexture(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, shadowMap, 0);

	glDrawBuffer(GL_NONE);
	glReadBuffer(GL_NONE);

	GLenum Status = glCheckFramebufferStatus(GL_FRAMEBUFFER);

	if (Status != GL_FRAMEBUFFER_COMPLETE)
	{
		printf("Framebuffer error: %s\n", Status);
		return false;
	}

	return true;
}

void OmniShadowMap::Write()
{
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, FBO);
}

void OmniShadowMap::Read(GLenum texUnit)
{
	glActiveTexture(texUnit);
	glBindTexture(GL_TEXTURE_CUBE_MAP, shadowMap);
}

OmniShadowMap::~OmniShadowMap()
{
}
```
- Geometry Shader
```C++
#version 330
layout (triangles) in;
layout (triangle_strip, max_vertices=18) out;

uniform mat4 lightMatrices[6];
out vec4 FragPos;

void main()
{
	for(int face = 0; face < 6; ++face)
	{
		gl_Layer = face;
		for(int i = 0; i < 3; i++)
		{
			FragPos = gl_in[i].gl_Position;
			gl_Position = lightMatrices[face] * FragPos;
			EmitVertex();
		}
		EndPrimitive();
	}
}
```
- Fragment Shader
``` C++
#version 330
in vec4 FragPos;

uniform vec3 lightPos;
uniform float farPlane;

void main()
{
	float distance = length(FragPos.xyz - lightPos);
	distance = distance/farPlane;
	gl_FragDepth = distance;
}
```
## Skybox
![opengl_skybox](https://user-images.githubusercontent.com/96270683/188812882-16438cf6-2ea3-4d53-aee6-9c07f2b2b3a2.PNG)

