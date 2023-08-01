# 3D Rendering Program

![opengl_final](https://user-images.githubusercontent.com/96270683/188821931-f96d2f21-9546-48a2-9651-fef8c7cb5d18.PNG)
## Introduction
This is a 3D rendering program implemented using OpenGL library.


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
		
## Camera projection
Projects the camera into perspective. Calculate the final position of the vertex by computing projection matrix * view space matrix * world space matrix.



![opengl_projection](https://user-images.githubusercontent.com/96270683/188787274-6c6570e3-9cf4-43d0-acc4-535d8760e7ab.PNG)
``` c++
model = glm::mat4(1.0f);
model = glm::translate(model, glm::vec3(0.0f, 1.0f, -2.5f));
model = glm::scale(model, glm::vec3(0.4f, 0.4f, 1.0f));

//Create camera
camera = Camera(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f), -90.0f, 0.0f, 5.0f, 0.5f);

//Set projection data (fov: 60 degrees, aspect ratio: screen width/length, near plane: 0.1f, far plane: 100.0)
glm::mat4 projection = glm::perspective(glm::radians(60.0f), (GLfloat)bufferWidth / (GLfloat)bufferHeight, 0.1f, 100.0f);

//Pass model(world space), projection(projection), view(view space) values ​​to vertex shader
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
	//Calculate the final position of the vertex by computing the projection matrix (projection) * view space matrix (view) * world space matrix (model).
	gl_Position = projection * view * model * vec4(pos, 1.0);
	vCol = vec4(clamp(pos, 0.0f, 1.0f), 1.0f);
}
```
## Texture mapping
Load the texture file and apply it to the model.


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

	/**Set texture mapping options*/
	//Apply GL_REPEAT option when the horizontal range is out of [0.0,1.0]
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
	//Apply the GL_REPEAT option when the vertical range is out of [0.0,1.0]
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
	//Apply GL_LINEAR option when polygon is smaller than texture
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	//Apply GL_LINEAR option when polygon is larger than texture
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

	//Set rendering options for loaded textures
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, texData);
	
	//Generate mipmap
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
## Lighting
Phong Lighting Model (Ambient + Diffuse + Specular) was used to implement lighting.
### Ambient


![lighting_ambient](https://user-images.githubusercontent.com/96270683/235663786-f9c6df61-591b-4d50-ab7c-0953911bbcab.png)
### Diffuse


![lighting_diffuse](https://user-images.githubusercontent.com/96270683/235663845-1bb6a82f-8359-4038-befd-6d80e18e1d6e.png)
### Specular


![lighting_speccular](https://user-images.githubusercontent.com/96270683/235663875-c334c1d8-7823-4ee2-8be7-8bb7490ca465.png)
### Directional Light
Ambient, diffuse, and specular light operations are processed in the fragment shader to implement directional light.


![opengl_directional_light](https://user-images.githubusercontent.com/96270683/188805453-cd1a67df-6daf-400a-8efc-c726738456e1.PNG)
- Fragment Shader
``` c++
void main()
{
	/**Ambient Light calculation (light color * ambient intensity)*/
	vec4 ambientColour = vec4(directionalLight.colour, 1.0f) * directionalLight.ambientIntensity;
	
	/**Diffuse Light calculation (light color * diffuse intensity * diffuse ratio)*/
	float diffuseFactor = max(dot(normalize(Normal), normalize(directionalLight.direction)), 0.0f); //Calculate diffuseFactor through normal vector and light direction dot product
	vec4 diffuseColour = vec4(directionalLight.colour, 1.0f) * directionalLight.diffuseIntensity * diffuseFactor;
	
	/**Specular Light calculation (light color * specular intensity * specular ratio with shininess applied)*/
	vec4 specularColour = vec4(0, 0, 0, 0);
	if(diffuseFactor > 0.0f)
	{
		vec3 fragToEye = normalize(eyePosition - FragPos);
		vec3 reflectedVertex = normalize(reflect(directionalLight.direction, normalize(Normal)));
		
		//Calculate the specularFactor through the dot product of the vector pointing from the fragment to the camera and the light reflection vector.
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
Point light is processed by adding attenuation calculation to directional light calculation.

![opengl_point_light](https://user-images.githubusercontent.com/96270683/188807781-70477324-3bf0-4606-a922-3633226c5802.PNG)
- Fragment Shader
``` c++
//Point light calculation (directional light operation + light attenuation calculation)
vec4 CalcPointLights()
{
	vec4 totalColour = vec4(0, 0, 0, 0);
	for(int i = 0; i < pointLightCount; i++)
	{
		vec3 direction = FragPos - pointLights[i].position;
		
		//Calculate distance from point light
		float distance = length(direction);
		
		//Direction from normalized point light
		direction = normalize(direction);
		
		//Result of directional light element
		vec4 colour = CalcLightByDirection(pointLights[i].base, direction);
		
		//Derivation of light attenuation value using quadratic equation
		float attenuation = pointLights[i].exponent * distance * distance +
							pointLights[i].linear * distance +
							pointLights[i].constant;
		
		totalColour += (colour / attenuation);
	}
	
	return totalColour;
}

//Calculate ambient, diffuse, and specular elements
vec4 CalcLightByDirection(Light light, vec3 direction)
{
	//Calculate ambient lighting
	vec4 ambientColour = vec4(light.colour, 1.0f) * light.ambientIntensity;
	
	//Calculate diffuse lighting
	float diffuseFactor = max(dot(normalize(Normal), normalize(direction)), 0.0f);
	vec4 diffuseColour = vec4(light.colour * light.diffuseIntensity * diffuseFactor, 1.0f);
	
	//Calculate specular lighting
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

	//Return the sum of all lighting operations
	return (ambientColour + diffuseColour + specularColour);
}

void main()
{
	//Apply directional light calculation result
	vec4 finalColour = CalcDirectionalLight();
	//Apply point lights calculation result
	finalColour += CalcPointLights();
	
	colour = texture(theTexture, TexCoord) * finalColour;
}
```
### Spot Light
Spot light implementation.


![opengl_spot_light](https://user-images.githubusercontent.com/96270683/188808820-5160caeb-7ccd-42e2-bf3b-7566f561c2e2.PNG)
``` c++
vec4 CalcSpotLight(SpotLight sLight)
{
	vec3 rayDirection = normalize(FragPos - sLight.base.position);
	float slFactor = dot(rayDirection, sLight.direction);
	
	//Process the light if it is within range of the Spot Light, otherwise ignore it.
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
	//The final color value is obtained by adding the calculated values ​​of directional light, point lights, and spot lights.
	vec4 finalColour = CalcDirectionalLight();
	finalColour += CalcPointLights();
	finalColour += CalcSpotLights();
	
	colour = texture(theTexture, TexCoord) * finalColour;
}
```
## Loading models


![opengl_model](https://user-images.githubusercontent.com/96270683/188810385-fb06b19d-3115-49dc-a741-64e09035da52.PNG)
``` c++
void Model::LoadModel(const std::string & fileName)
{
	//Load model data from file
	Assimp::Importer importer;
	const aiScene *scene = importer.ReadFile(fileName, aiProcess_Triangulate | aiProcess_FlipUVs | aiProcess_GenSmoothNormals | aiProcess_JoinIdenticalVertices);

	if (!scene)
	{
		printf("Model (%s) failed to load: %s", fileName, importer.GetErrorString());
		return;
	}
	
	//Start loading from the root node
	LoadNode(scene->mRootNode, scene);
	//Load materials
	LoadMaterials(scene);
}

void Model::LoadNode(aiNode * node, const aiScene * scene)
{
	//Load a mesh belonging to a node
	for (size_t i = 0; i < node->mNumMeshes; i++)
	{
		LoadMesh(scene->mMeshes[node->mMeshes[i]], scene);
	}
	
	//Continue loading child nodes in a recursive fashion
	for (size_t i = 0; i < node->mNumChildren; i++)
	{
		LoadNode(node->mChildren[i], scene);
	}
}

void Model::LoadMesh(aiMesh * mesh, const aiScene * scene)
{
	std::vector<GLfloat> vertices;
	std::vector<unsigned int> indices;

	//Save vertex data
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
	
	//Save index data
	for (size_t i = 0; i < mesh->mNumFaces; i++)
	{
		aiFace face = mesh->mFaces[i];
		for (size_t j = 0; j < face.mNumIndices; j++)
		{
			indices.push_back(face.mIndices[j]);
		}
	}

	//Create mesh
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
		
		//Render texture
		if (materialIndex < textureList.size() && textureList[materialIndex])
		{
			textureList[materialIndex]->UseTexture();
		}

		//Render mesh
		meshList[i]->RenderMesh();
	}
}
```
## Shadow Map
### Directional Shadow Map
Used to process shadows for directional lights.

![shadow_mapping_theory_spaces](https://user-images.githubusercontent.com/96270683/236655201-5bbc73e1-525b-49b4-9616-3a962195a6c6.png)
![opengl_directional_light](https://user-images.githubusercontent.com/96270683/188812990-fb3984b6-cf9e-4c11-860b-9ef2eaf276a2.PNG)
- ShadowMap.cpp
``` c++
bool ShadowMap::Init(unsigned int width, unsigned int height)
{
	shadowWidth = width; shadowHeight = height;

	glGenFramebuffers(1, &FBO);

	//Set the shadow map texture
	glGenTextures(1, &shadowMap);
	glBindTexture(GL_TEXTURE_2D, shadowMap);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_DEPTH_COMPONENT, width, height, 0, GL_DEPTH_COMPONENT, GL_FLOAT, nullptr);
	
	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);
	
	float borderColor[] = { 1.0f, 1.0f, 1.0f, 1.0f };
	glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);

	//Store shadow map data in framebuffer
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
uniform mat4 directionalLightTransform; //Transformation matrix to position of light source

void main()
{
	gl_Position = directionalLightTransform * model * vec4(pos, 1.0);
}
```
### Omni Directional Shadow Map
Used for shadow processing for Point Light and Spot Light


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

