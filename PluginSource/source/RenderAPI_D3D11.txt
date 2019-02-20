#include "RenderAPI.h"
#include "PlatformBase.h"

// Direct3D 11 implementation of RenderAPI.

#if SUPPORT_D3D11
#pragma comment(lib, "d3d11.lib")
#pragma comment(lib, "d3dx11.lib")
#pragma comment(lib, "d3dx10.lib")
#pragma comment(lib, "dxgi.lib")
#include <assert.h>
#include <d3d11.h>
#include "Unity/IUnityGraphicsD3D11.h"
#include <d3dx11.h>
#include <D3DX10.h>
#include <xnamath.h>

class RenderAPI_D3D11 : public RenderAPI
{
public:
	RenderAPI_D3D11();
	virtual ~RenderAPI_D3D11() { }

	virtual void ProcessDeviceEvent(UnityGfxDeviceEventType type, IUnityInterfaces* interfaces);

	virtual bool GetUsesReverseZ() { return (int)m_Device->GetFeatureLevel() >= (int)D3D_FEATURE_LEVEL_10_0; }

	virtual void DrawSimpleTriangles(const float worldMatrix[16], int triangleCount, const void* verticesFloat3Byte4);

	virtual void* BeginModifyTexture(void* textureHandle, int textureWidth, int textureHeight, int* outRowPitch);
	virtual void EndModifyTexture(void* textureHandle, int textureWidth, int textureHeight, int rowPitch, void* dataPtr);

	virtual void* BeginModifyVertexBuffer(void* bufferHandle, size_t* outBufferSize);
	virtual void EndModifyVertexBuffer(void* bufferHandle);
	virtual	void SetRenderTexture(void* colorBuffer, void* depthBuffer, int textureWidth, int textureHeight) ;

	virtual void RenderDistrutionPass();
private:
	void CreateResources();
	void ReleaseResources();

private:
	ID3D11Device* m_Device;
	ID3D11Buffer* m_VB; // vertex buffer
	ID3D11Buffer* m_CB; // constant buffer
	ID3D11VertexShader* m_VertexShader;
	ID3D11PixelShader* m_PixelShader;
	ID3D11InputLayout* m_InputLayout;
	ID3D11RasterizerState* m_RasterState;
	ID3D11BlendState* m_BlendState;
	ID3D11DepthStencilState* m_DepthState;

	//---------------20181220---------------------//
	D3DXMATRIX m_projectionMatrix;
	D3DXMATRIX m_worldMatrix;
	D3DXMATRIX m_orthoMatrix;
	D3DXMATRIX m_viewMatrix;

	ID3D11Buffer *m_vertexBuffer, *m_indexBuffer;
	int m_vertexCount, m_indexCount;
	ID3D11ShaderResourceView* m_Texture;
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11Buffer* m_matrixBuffer;
	ID3D11SamplerState* m_sampleState;

	float fieldOfView, screenAspect;



	struct VertexType
	{
		D3DXVECTOR3 position;
		D3DXVECTOR2 texture;
	};
	struct MatrixBufferType
	{
		D3DXMATRIX world;
		D3DXMATRIX view;
		D3DXMATRIX projection;
	};

	D3D11_MAPPED_SUBRESOURCE mappedResource;
	MatrixBufferType* dataPtr;
	unsigned int bufferNumber;


	ID3D11Texture2D *  pcolorBuffer = NULL;
	ID3D11Texture2D *  pdepthBuffer = NULL;

	ID3D11Texture2D *pTexture = NULL;
};


RenderAPI* CreateRenderAPI_D3D11()
{
	return new RenderAPI_D3D11();
}


// Simple compiled shader bytecode.
//
// Shader source that was used:
#if 0
cbuffer MyCB : register(b0)
{
	float4x4 worldMatrix;
}
void VS(float3 pos : POSITION, float4 color : COLOR, out float4 ocolor : COLOR, out float4 opos : SV_Position)
{
	opos = mul(worldMatrix, float4(pos, 1));
	ocolor = color;
}
float4 PS(float4 color : COLOR) : SV_TARGET
{
	return color;
}
#endif // #if 0
//
// Which then was compiled with:
// fxc /Tvs_4_0_level_9_3 /EVS source.hlsl /Fh outVS.h /Qstrip_reflect /Qstrip_debug /Qstrip_priv
// fxc /Tps_4_0_level_9_3 /EPS source.hlsl /Fh outPS.h /Qstrip_reflect /Qstrip_debug /Qstrip_priv
// and results pasted & formatted to take less lines here
const BYTE kVertexShaderCode[] =
{
	68,88,66,67,86,189,21,50,166,106,171,1,10,62,115,48,224,137,163,129,1,0,0,0,168,2,0,0,4,0,0,0,48,0,0,0,0,1,0,0,4,2,0,0,84,2,0,0,
	65,111,110,57,200,0,0,0,200,0,0,0,0,2,254,255,148,0,0,0,52,0,0,0,1,0,36,0,0,0,48,0,0,0,48,0,0,0,36,0,1,0,48,0,0,0,0,0,
	4,0,1,0,0,0,0,0,0,0,0,0,1,2,254,255,31,0,0,2,5,0,0,128,0,0,15,144,31,0,0,2,5,0,1,128,1,0,15,144,5,0,0,3,0,0,15,128,
	0,0,85,144,2,0,228,160,4,0,0,4,0,0,15,128,1,0,228,160,0,0,0,144,0,0,228,128,4,0,0,4,0,0,15,128,3,0,228,160,0,0,170,144,0,0,228,128,
	2,0,0,3,0,0,15,128,0,0,228,128,4,0,228,160,4,0,0,4,0,0,3,192,0,0,255,128,0,0,228,160,0,0,228,128,1,0,0,2,0,0,12,192,0,0,228,128,
	1,0,0,2,0,0,15,224,1,0,228,144,255,255,0,0,83,72,68,82,252,0,0,0,64,0,1,0,63,0,0,0,89,0,0,4,70,142,32,0,0,0,0,0,4,0,0,0,
	95,0,0,3,114,16,16,0,0,0,0,0,95,0,0,3,242,16,16,0,1,0,0,0,101,0,0,3,242,32,16,0,0,0,0,0,103,0,0,4,242,32,16,0,1,0,0,0,
	1,0,0,0,104,0,0,2,1,0,0,0,54,0,0,5,242,32,16,0,0,0,0,0,70,30,16,0,1,0,0,0,56,0,0,8,242,0,16,0,0,0,0,0,86,21,16,0,
	0,0,0,0,70,142,32,0,0,0,0,0,1,0,0,0,50,0,0,10,242,0,16,0,0,0,0,0,70,142,32,0,0,0,0,0,0,0,0,0,6,16,16,0,0,0,0,0,
	70,14,16,0,0,0,0,0,50,0,0,10,242,0,16,0,0,0,0,0,70,142,32,0,0,0,0,0,2,0,0,0,166,26,16,0,0,0,0,0,70,14,16,0,0,0,0,0,
	0,0,0,8,242,32,16,0,1,0,0,0,70,14,16,0,0,0,0,0,70,142,32,0,0,0,0,0,3,0,0,0,62,0,0,1,73,83,71,78,72,0,0,0,2,0,0,0,
	8,0,0,0,56,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,0,0,0,0,7,7,0,0,65,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,1,0,0,0,
	15,15,0,0,80,79,83,73,84,73,79,78,0,67,79,76,79,82,0,171,79,83,71,78,76,0,0,0,2,0,0,0,8,0,0,0,56,0,0,0,0,0,0,0,0,0,0,0,
	3,0,0,0,0,0,0,0,15,0,0,0,62,0,0,0,0,0,0,0,1,0,0,0,3,0,0,0,1,0,0,0,15,0,0,0,67,79,76,79,82,0,83,86,95,80,111,115,
	105,116,105,111,110,0,171,171
};



const BYTE kPixelShaderCode[]=
{
	68,88,66,67,196,65,213,199,14,78,29,150,87,236,231,156,203,125,244,112,1,0,0,0,32,1,0,0,4,0,0,0,48,0,0,0,124,0,0,0,188,0,0,0,236,0,0,0,
	65,111,110,57,68,0,0,0,68,0,0,0,0,2,255,255,32,0,0,0,36,0,0,0,0,0,36,0,0,0,36,0,0,0,36,0,0,0,36,0,0,0,36,0,1,2,255,255,
	31,0,0,2,0,0,0,128,0,0,15,176,1,0,0,2,0,8,15,128,0,0,228,176,255,255,0,0,83,72,68,82,56,0,0,0,64,0,0,0,14,0,0,0,98,16,0,3,
	242,16,16,0,0,0,0,0,101,0,0,3,242,32,16,0,0,0,0,0,54,0,0,5,242,32,16,0,0,0,0,0,70,30,16,0,0,0,0,0,62,0,0,1,73,83,71,78,
	40,0,0,0,1,0,0,0,8,0,0,0,32,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,0,0,0,0,15,15,0,0,67,79,76,79,82,0,171,171,79,83,71,78,
	44,0,0,0,1,0,0,0,8,0,0,0,32,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,0,0,0,0,15,0,0,0,83,86,95,84,65,82,71,69,84,0,171,171
};

const BYTE KVSCODE[] =
{
	68,88,66,67,-73,-13,-33,53,50,-44,-122,-105,-77,53,75,-21,-10,48,-108,118,1,0,0,0,32,5,0,0,5,0,0,0,52,0,0,0,
	-72,1,0,0,12,2,0,0,100,2,0,0,-124,4,0,0,82,68,69,70,124,1,0,0,1,0,0,0,108,0,0,0,1,0,0,0,60,0,0,0,0,5,-2,-1,0,9,0,
	0,72,1,0,0,82,68,49,49,60,0,0,0,24,0,0,0,32,0,0,0,40,0,0,0,36,0,0,0,12,0,0,0,0,0,0,0,92,0,0,0,0,0,0,0,0,0,0,0,0,0,
	0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,77,97,116,114,105,120,66,117,102,102,101,114,0,-85,-85,-85,92,0,0,0,3,0,
	0,0,-124,0,0,0,-64,0,0,0,0,0,0,0,0,0,0,0,-4,0,0,0,0,0,0,0,64,0,0,0,2,0,0,0,8,1,0,0,0,0,0,0,-1,-1,-1,-1,0,0,0,0,-1,-1,
	-1,-1,0,0,0,0,44,1,0,0,64,0,0,0,64,0,0,0,2,0,0,0,8,1,0,0,0,0,0,0,-1,-1,-1,-1,0,0,0,0,-1,-1,-1,-1,0,0,0,0,55,1,0,0,
	-128,0,0,0,64,0,0,0,2,0,0,0,8,1,0,0,0,0,0,0,-1,-1,-1,-1,0,0,0,0,-1,-1,-1,-1,0,0,0,0,119,111,114,108,100,77,97,
	116,114,105,120,0,3,0,3,0,4,0,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,118,105,101,119,77,
	97,116,114,105,120,0,112,114,111,106,101,99,116,105,111,110,77,97,116,114,105,120,0,77,105,99,114,
	111,115,111,102,116,32,40,82,41,32,72,76,83,76,32,83,104,97,100,101,114,32,67,111,109,112,105,108,
	101,114,32,57,46,50,57,46,57,53,50,46,51,49,49,49,0,-85,-85,-85,73,83,71,78,76,0,0,0,2,0,0,0,8,0,0,0,56,
	0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,0,0,0,0,15,7,0,0,65,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,1,0,0,0,3,3,0,0,80,79,83,73,84,73,
	79,78,0,84,69,88,67,79,79,82,68,0,-85,-85,79,83,71,78,80,0,0,0,2,0,0,0,8,0,0,0,56,0,0,0,0,0,0,0,1,0,0,0,3,0,0,
	0,0,0,0,0,15,0,0,0,68,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,1,0,0,0,3,12,0,0,83,86,95,80,79,83,73,84,73,79,78,0,84,69,
	88,67,79,79,82,68,0,-85,-85,-85,83,72,69,88,24,2,0,0,80,0,1,0,-122,0,0,0,106,8,0,1,89,0,0,4,70,-114,32,0,0,
	0,0,0,12,0,0,0,95,0,0,3,114,16,16,0,0,0,0,0,95,0,0,3,50,16,16,0,1,0,0,0,103,0,0,4,-14,32,16,0,0,0,0,0,1,0,0,0,
	101,0,0,3,50,32,16,0,1,0,0,0,104,0,0,2,2,0,0,0,54,0,0,5,114,0,16,0,0,0,0,0,70,18,16,0,0,0,0,0,54,0,0,5,-126,0,
	16,0,0,0,0,0,1,64,0,0,0,0,-128,63,17,0,0,8,18,0,16,0,1,0,0,0,70,14,16,0,0,0,0,0,70,-114,32,0,0,0,0,0,0,0,0,0,
	17,0,0,8,34,0,16,0,1,0,0,0,70,14,16,0,0,0,0,0,70,-114,32,0,0,0,0,0,1,0,0,0,17,0,0,8,66,0,16,0,1,0,0,0,70,14,
	16,0,0,0,0,0,70,-114,32,0,0,0,0,0,2,0,0,0,17,0,0,8,-126,0,16,0,1,0,0,0,70,14,16,0,0,0,0,0,70,-114,32,0,0,0,0,
	0,3,0,0,0,17,0,0,8,18,0,16,0,0,0,0,0,70,14,16,0,1,0,0,0,70,-114,32,0,0,0,0,0,4,0,0,0,17,0,0,8,34,0,16,0,0,0,0,
	0,70,14,16,0,1,0,0,0,70,-114,32,0,0,0,0,0,5,0,0,0,17,0,0,8,66,0,16,0,0,0,0,0,70,14,16,0,1,0,0,0,70,-114,32,0,
	0,0,0,0,6,0,0,0,17,0,0,8,-126,0,16,0,0,0,0,0,70,14,16,0,1,0,0,0,70,-114,32,0,0,0,0,0,7,0,0,0,17,0,0,8,18,32,
	16,0,0,0,0,0,70,14,16,0,0,0,0,0,70,-114,32,0,0,0,0,0,8,0,0,0,17,0,0,8,34,32,16,0,0,0,0,0,70,14,16,0,0,0,0,0,70,
	-114,32,0,0,0,0,0,9,0,0,0,17,0,0,8,66,32,16,0,0,0,0,0,70,14,16,0,0,0,0,0,70,-114,32,0,0,0,0,0,10,0,0,0,17,0,0,8,
	-126,32,16,0,0,0,0,0,70,14,16,0,0,0,0,0,70,-114,32,0,0,0,0,0,11,0,0,0,54,0,0,5,50,32,16,0,1,0,0,0,70,16,16,0,
	1,0,0,0,62,0,0,1,83,84,65,84,-108,0,0,0,16,0,0,0,2,0,0,0,0,0,0,0,4,0,0,0,12,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,
	0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
	0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
};

const BYTE KPSCODE[] =
{
  68,88,66,67,47,115,109,-121,-115,18,37,63,65,-84,-53,100,-39,-86,-103,-30,1,0,0,0,-92,2,0,0,5,0,0,0,52,0,0,0,
  4,1,0,0,92,1,0,0,-112,1,0,0,8,2,0,0,82,68,69,70,-56,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,60,0,0,0,0,5,-1,-1,0,9,0,0,-107,0,
  0,0,82,68,49,49,60,0,0,0,24,0,0,0,32,0,0,0,40,0,0,0,36,0,0,0,12,0,0,0,0,0,0,0,124,0,0,0,3,0,0,0,0,0,0,0,0,0,0,0,0,0,
  0,0,0,0,0,0,1,0,0,0,0,0,0,0,-121,0,0,0,2,0,0,0,5,0,0,0,4,0,0,0,-1,-1,-1,-1,0,0,0,0,1,0,0,0,12,0,0,0,83,97,109,112,108,
  101,84,121,112,101,0,115,104,97,100,101,114,84,101,120,116,117,114,101,0,77,105,99,114,111,115,111,102,
  116,32,40,82,41,32,72,76,83,76,32,83,104,97,100,101,114,32,67,111,109,112,105,108,101,114,32,57,46,50,57,
  46,57,53,50,46,51,49,49,49,0,-85,-85,73,83,71,78,80,0,0,0,2,0,0,0,8,0,0,0,56,0,0,0,0,0,0,0,1,0,0,0,3,0,0,0,0,0,0,0,
  15,0,0,0,68,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,1,0,0,0,3,3,0,0,83,86,95,80,79,83,73,84,73,79,78,0,84,69,88,67,79,79,82,
  68,0,-85,-85,-85,79,83,71,78,44,0,0,0,1,0,0,0,8,0,0,0,32,0,0,0,0,0,0,0,0,0,0,0,3,0,0,0,0,0,0,0,15,0,0,0,83,86,95,84,
  65,82,71,69,84,0,-85,-85,83,72,69,88,112,0,0,0,80,0,0,0,28,0,0,0,106,8,0,1,90,0,0,3,0,96,16,0,0,0,0,0,88,24,0,4,0,
  112,16,0,0,0,0,0,85,85,0,0,98,16,0,3,50,16,16,0,1,0,0,0,101,0,0,3,-14,32,16,0,0,0,0,0,69,0,0,-117,-62,0,0,-128,67,
  85,21,0,-14,32,16,0,0,0,0,0,70,16,16,0,1,0,0,0,70,126,16,0,0,0,0,0,0,96,16,0,0,0,0,0,62,0,0,1,83,84,65,84,-108,0,0,
  0,2,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,
  0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
  0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
};
RenderAPI_D3D11::RenderAPI_D3D11()
	: m_Device(NULL)
	, m_VB(NULL)
	, m_CB(NULL)
	, m_VertexShader(NULL)
	, m_PixelShader(NULL)
	, m_InputLayout(NULL)
	, m_RasterState(NULL)
	, m_BlendState(NULL)
	, m_DepthState(NULL)
{
}


void RenderAPI_D3D11::ProcessDeviceEvent(UnityGfxDeviceEventType type, IUnityInterfaces* interfaces)
{
	switch (type)
	{
	case kUnityGfxDeviceEventInitialize:
	{
		IUnityGraphicsD3D11* d3d = interfaces->Get<IUnityGraphicsD3D11>();
		m_Device = d3d->GetDevice();
		CreateResources();
		break;
	}
	case kUnityGfxDeviceEventShutdown:
		ReleaseResources();
		break;
	}
}


void RenderAPI_D3D11::CreateResources()
{
	D3D11_BUFFER_DESC desc;
	memset(&desc, 0, sizeof(desc));

	// vertex buffer
	desc.Usage = D3D11_USAGE_DEFAULT;
	desc.ByteWidth = 1024;
	desc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
	m_Device->CreateBuffer(&desc, NULL, &m_VB);

	// constant buffer
	desc.Usage = D3D11_USAGE_DEFAULT;
	desc.ByteWidth = 64; // hold 1 matrix
	desc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
	desc.CPUAccessFlags = 0;
	m_Device->CreateBuffer(&desc, NULL, &m_CB);

	// shaders
	HRESULT hr;
	hr = m_Device->CreateVertexShader(kVertexShaderCode, sizeof(kVertexShaderCode), nullptr, &m_VertexShader);
	if (FAILED(hr))
		OutputDebugStringA("Failed to create vertex shader.\n");
	hr = m_Device->CreatePixelShader(kPixelShaderCode, sizeof(kPixelShaderCode), nullptr, &m_PixelShader);
	if (FAILED(hr))
		OutputDebugStringA("Failed to create pixel shader.\n");

	// input layout
	if (m_VertexShader)
	{
		D3D11_INPUT_ELEMENT_DESC s_DX11InputElementDesc[] =
		{
			{ "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA, 0 },
			{ "COLOR", 0, DXGI_FORMAT_R8G8B8A8_UNORM, 0, 12, D3D11_INPUT_PER_VERTEX_DATA, 0 },
		};
		m_Device->CreateInputLayout(s_DX11InputElementDesc, 2, kVertexShaderCode, sizeof(kVertexShaderCode), &m_InputLayout);
	}

	// render states
	D3D11_RASTERIZER_DESC rsdesc;
	memset(&rsdesc, 0, sizeof(rsdesc));
	rsdesc.FillMode = D3D11_FILL_SOLID;
	rsdesc.CullMode = D3D11_CULL_NONE;
	rsdesc.DepthClipEnable = TRUE;
	m_Device->CreateRasterizerState(&rsdesc, &m_RasterState);

	D3D11_DEPTH_STENCIL_DESC dsdesc;
	memset(&dsdesc, 0, sizeof(dsdesc));
	dsdesc.DepthEnable = TRUE;
	dsdesc.DepthWriteMask = D3D11_DEPTH_WRITE_MASK_ZERO;
	dsdesc.DepthFunc = GetUsesReverseZ() ? D3D11_COMPARISON_GREATER_EQUAL : D3D11_COMPARISON_LESS_EQUAL;
	m_Device->CreateDepthStencilState(&dsdesc, &m_DepthState);

	D3D11_BLEND_DESC bdesc;
	memset(&bdesc, 0, sizeof(bdesc));
	bdesc.RenderTarget[0].BlendEnable = FALSE;
	bdesc.RenderTarget[0].RenderTargetWriteMask = 0xF;
	m_Device->CreateBlendState(&bdesc, &m_BlendState);

	//----------20181220--------------//
	 
	
	float Width = 1920;
	float	Height = 1080;

	// Setup the projection matrix.
	fieldOfView = (float)D3DX_PI / 4.0f;
	screenAspect = (float)Width / (float)Height;

	// Create the projection matrix for 3D rendering.
	D3DXMatrixPerspectiveFovLH(&m_projectionMatrix, fieldOfView, screenAspect, 0.01F, 1000.0F);

	// Initialize the world matrix to the identity matrix.
	D3DXMatrixIdentity(&m_worldMatrix);

	// Create an orthographic projection matrix for 2D rendering.
	D3DXMatrixOrthoLH(&m_orthoMatrix, (float)Width, (float)Height, 0.01F, 1000.0F);


	VertexType* vertices;
	unsigned long* indices;
	D3D11_BUFFER_DESC vertexBufferDesc, indexBufferDesc;
	D3D11_SUBRESOURCE_DATA vertexData, indexData;



	// Set the number of vertices in the vertex array.
	m_vertexCount = 4;

	// Set the number of indices in the index array.
	m_indexCount = 6;

	// Create the vertex array.
	vertices = new VertexType[m_vertexCount];
	if (!vertices)
	{
		OutputDebugStringA("Failed to create vertices .\n");
	}

	// Create the index array.
	indices = new unsigned long[m_indexCount];
	if (!indices)
	{
		OutputDebugStringA("Failed to create indices .\n");
	}

	// Load the vertex array with data.
	vertices[0].position = D3DXVECTOR3(-1.0f, -1.0f, 0.0f);  // Bottom left.
	vertices[0].texture = D3DXVECTOR2(0.0f, 1.0f);

	vertices[1].position = D3DXVECTOR3(-1.0f, 1.0f, 0.0f);  // Top middle.
	vertices[1].texture = D3DXVECTOR2(0.0f, 0.0f);

	vertices[2].position = D3DXVECTOR3(1.0f, 1.0f, 0.0f);  // Bottom right.
	vertices[2].texture = D3DXVECTOR2(1.0f, 0.0f);

	vertices[3].position = D3DXVECTOR3(1.0f, -1.0f, 0.0f);  // Bottom right.
	vertices[3].texture = D3DXVECTOR2(1.0f, 1.0f);

	 

	// Load the index array with data.
	indices[0] = 0;  // Bottom left.
	indices[1] = 1;  // Top middle.
	indices[2] = 2;  // Bottom right.
	indices[3] = 0;  // Bottom right.
	indices[4] = 2;  // Bottom right.
	indices[5] = 3;  // Bottom right.

					 // Set up the description of the static vertex buffer.
	vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	vertexBufferDesc.ByteWidth = sizeof(VertexType) * m_vertexCount;
	vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
	vertexBufferDesc.CPUAccessFlags = 0;
	vertexBufferDesc.MiscFlags = 0;
	vertexBufferDesc.StructureByteStride = 0;

	// Give the subresource structure a pointer to the vertex data.
	vertexData.pSysMem = vertices;
	vertexData.SysMemPitch = 0;
	vertexData.SysMemSlicePitch = 0;
 
	// Now create the vertex buffer.
	hr = m_Device->CreateBuffer(&vertexBufferDesc, &vertexData, &m_vertexBuffer);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to create  vertex buffer .\n");
	}

	// Set up the description of the static index buffer.
	indexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	indexBufferDesc.ByteWidth = sizeof(unsigned long) * m_indexCount;
	indexBufferDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;
	indexBufferDesc.CPUAccessFlags = 0;
	indexBufferDesc.MiscFlags = 0;
	indexBufferDesc.StructureByteStride = 0;

	// Give the subresource structure a pointer to the index data.
	indexData.pSysMem = indices;
	indexData.SysMemPitch = 0;
	indexData.SysMemSlicePitch = 0;

	// Create the index buffer.
	hr = m_Device->CreateBuffer(&indexBufferDesc, &indexData, &m_indexBuffer);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to create  index buffer .\n");
	}

	// Release the arrays now that the vertex and index buffers have been created and loaded.
	delete[] vertices;
	vertices = 0;

	delete[] indices;
	indices = 0;

	//// Load the texture in.
	//hr = D3DX11CreateShaderResourceViewFromFile(m_Device, TEXT("D:/1.jpg"), NULL, NULL, &m_Texture, NULL);
	//if (FAILED(hr))
	//{
	//	OutputDebugStringA("Failed to Load the texture D:/1.jpg.\n");
	//}



	ID3D10Blob* errorMessage;
	ID3D10Blob* vertexShaderBuffer;
	ID3D10Blob* pixelShaderBuffer;
	D3D11_INPUT_ELEMENT_DESC polygonLayout[2];
	unsigned int numElements;
	D3D11_BUFFER_DESC matrixBufferDesc;
	D3D11_SAMPLER_DESC samplerDesc;


	// Initialize the pointers this function will use to null.
	errorMessage = 0;
	//vertexShaderBuffer = 0;
	//pixelShaderBuffer = 0;

	//// Compile the vertex shader code.
	//hr = D3DX11CompileFromFile(TEXT("D:/texture.vs"), NULL, NULL, "TextureVertexShader", "vs_5_0", D3D10_SHADER_ENABLE_STRICTNESS, 0, NULL,
	//	&vertexShaderBuffer, &errorMessage, NULL);
	//if (FAILED(hr))
	//{
	//	// If the shader failed to compile it should have writen something to the error message.
	//	OutputDebugStringA("Failed to  Compile the vertex shader code D:/texture.vs.\n");
 //
	//}

	//// Compile the pixel shader code.
	//hr = D3DX11CompileFromFile(TEXT("D:/texture.ps"), NULL, NULL, "TexturePixelShader", "ps_5_0", D3D10_SHADER_ENABLE_STRICTNESS, 0, NULL,
	//	&pixelShaderBuffer, &errorMessage, NULL);
	//if (FAILED(hr))
	//{
	//	OutputDebugStringA("Failed to  Compile the pixel shader code D:/texture.ps.\n");

	//}

	//// Create the vertex shader from the buffer.
	//hr = m_Device->CreateVertexShader(vertexShaderBuffer->GetBufferPointer(), vertexShaderBuffer->GetBufferSize(), NULL, &m_vertexShader);
	//if (FAILED(hr))
	//{
	//	OutputDebugStringA("Failed to  Create the vertex shader from the buffer.\n");

	//	 
	//}

	//// Create the pixel shader from the buffer.
	//hr = m_Device->CreatePixelShader(pixelShaderBuffer->GetBufferPointer(), pixelShaderBuffer->GetBufferSize(), NULL, &m_pixelShader);
	//if (FAILED(hr))
	//{
	//	OutputDebugStringA("Failed to  Create the pixel shader from the buffer.\n");

	//}

	// Create the vertex shader from the buffer.
	hr = m_Device->CreateVertexShader(KVSCODE, sizeof(KVSCODE), NULL, &m_vertexShader);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to  Create the vertex shader from the buffer.\n");


	}

	// Create the pixel shader from the buffer.
	hr = m_Device->CreatePixelShader(KPSCODE, sizeof(KPSCODE), NULL, &m_pixelShader);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to  Create the pixel shader from the buffer.\n");

	}
 
	// Create the vertex input layout description.
	// This setup needs to match the VertexType stucture in the ModelClass and in the shader.
	polygonLayout[0].SemanticName = "POSITION";
	polygonLayout[0].SemanticIndex = 0;
	polygonLayout[0].Format = DXGI_FORMAT_R32G32B32_FLOAT;
	polygonLayout[0].InputSlot = 0;
	polygonLayout[0].AlignedByteOffset = 0;
	polygonLayout[0].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
	polygonLayout[0].InstanceDataStepRate = 0;

	polygonLayout[1].SemanticName = "TEXCOORD";
	polygonLayout[1].SemanticIndex = 0;
	polygonLayout[1].Format = DXGI_FORMAT_R32G32_FLOAT;
	polygonLayout[1].InputSlot = 0;
	polygonLayout[1].AlignedByteOffset = D3D11_APPEND_ALIGNED_ELEMENT;
	polygonLayout[1].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
	polygonLayout[1].InstanceDataStepRate = 0;

	// Get a count of the elements in the layout.
	numElements = sizeof(polygonLayout) / sizeof(polygonLayout[0]);

	// Create the vertex input layout.
	hr = m_Device->CreateInputLayout(polygonLayout, numElements, KVSCODE, sizeof(KVSCODE),
		&m_layout);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to Create the vertex input layout.\n");

		 
	}

	// Release the vertex shader buffer and pixel shader buffer since they are no longer needed.
	//vertexShaderBuffer->Release();
	//vertexShaderBuffer = 0;

	//pixelShaderBuffer->Release();
	//pixelShaderBuffer = 0;

	// Setup the description of the dynamic matrix constant buffer that is in the vertex shader.
	matrixBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
	matrixBufferDesc.ByteWidth = sizeof(MatrixBufferType);
	matrixBufferDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
	matrixBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
	matrixBufferDesc.MiscFlags = 0;
	matrixBufferDesc.StructureByteStride = 0;

	// Create the constant buffer pointer so we can access the vertex shader constant buffer from within this class.
	hr = m_Device->CreateBuffer(&matrixBufferDesc, NULL, &m_matrixBuffer);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to Create the constant buffer pointer so we can access the vertex shader constant buffer from within this class.\n");

		 
	}
	// Create a texture sampler state description.
	samplerDesc.Filter = D3D11_FILTER_MIN_MAG_MIP_LINEAR;
	samplerDesc.AddressU = D3D11_TEXTURE_ADDRESS_WRAP;
	samplerDesc.AddressV = D3D11_TEXTURE_ADDRESS_WRAP;
	samplerDesc.AddressW = D3D11_TEXTURE_ADDRESS_WRAP;
	samplerDesc.MipLODBias = 0.0f;
	samplerDesc.MaxAnisotropy = 1;
	samplerDesc.ComparisonFunc = D3D11_COMPARISON_ALWAYS;
	samplerDesc.BorderColor[0] = 0;
	samplerDesc.BorderColor[1] = 0;
	samplerDesc.BorderColor[2] = 0;
	samplerDesc.BorderColor[3] = 0;
	samplerDesc.MinLOD = 0;
	samplerDesc.MaxLOD = D3D11_FLOAT32_MAX;

	// Create the texture sampler state.
	hr = m_Device->CreateSamplerState(&samplerDesc, &m_sampleState);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to Create the texture sampler state..\n");

	}
	D3DXVECTOR3 up, position, lookAt;
	float yaw, pitch, roll;
	D3DXMATRIX rotationMatrix;


	// Setup the vector that points upwards.
	up.x = 0.0f;
	up.y = 1.0f;
	up.z = 0.0f;

	// Setup the position of the camera in the world.
	position.x = 0;
	position.y = 0;
	position.z = -10.0f;

	// Setup where the camera is looking by default.
	lookAt.x = 0.0f;
	lookAt.y = 0.0f;
	lookAt.z = 1.0f;

	// Set the yaw (Y axis), pitch (X axis), and roll (Z axis) rotations in radians.
	pitch = 0 * 0.0174532925f;
	yaw = 0 * 0.0174532925f;
	roll = 0 * 0.0174532925f;

	// Create the rotation matrix from the yaw, pitch, and roll values.
	D3DXMatrixRotationYawPitchRoll(&rotationMatrix, yaw, pitch, roll);

	// Transform the lookAt and up vector by the rotation matrix so the view is correctly rotated at the origin.
	D3DXVec3TransformCoord(&lookAt, &lookAt, &rotationMatrix);
	D3DXVec3TransformCoord(&up, &up, &rotationMatrix);

	// Translate the rotated camera position to the location of the viewer.
	lookAt = position + lookAt;

	// Finally create the view matrix from the three updated vectors.
	D3DXMatrixLookAtLH(&m_viewMatrix, &position, &lookAt, &up);




	// Transpose the matrices to prepare them for the shader.
	D3DXMatrixTranspose(&m_worldMatrix, &m_worldMatrix);
	D3DXMatrixTranspose(&m_viewMatrix, &m_viewMatrix);
	D3DXMatrixTranspose(&m_projectionMatrix, &m_projectionMatrix);

	// Lock the constant buffer so it can be written to.
	ID3D11DeviceContext* ctx = NULL;
	m_Device->GetImmediateContext(&ctx);
	hr = ctx->Map(m_matrixBuffer, 0, D3D11_MAP_WRITE_DISCARD, 0, &mappedResource);
	if (FAILED(hr))
	{
		OutputDebugStringA("Failed to Lock the constant buffer so it can be written to.\n");

	}

	// Get a pointer to the data in the constant buffer.
	dataPtr = (MatrixBufferType*)mappedResource.pData;

	// Copy the matrices into the constant buffer.
	dataPtr->world = m_worldMatrix;
	dataPtr->view = m_viewMatrix;
	dataPtr->projection = m_projectionMatrix;

	// Unlock the constant buffer.
	ctx->Unmap(m_matrixBuffer, 0);

	// Set the position of the constant buffer in the vertex shader.
	bufferNumber = 0;


	D3D11_TEXTURE2D_DESC mdesc;
	ZeroMemory(&mdesc, sizeof(mdesc));
	mdesc.Width = 1920;
	mdesc.Height = 1080;
 
	mdesc.BindFlags = D3D11_BIND_RENDER_TARGET | D3D11_BIND_SHADER_RESOURCE;
	mdesc.MiscFlags = D3D11_RESOURCE_MISC_SHARED; // This texture will be shared
											  // A DirectX 11 texture with D3D11_RESOURCE_MISC_SHARED_KEYEDMUTEX is not compatible with DirectX 9
											  // so a general named mutex is used for all texture types
	mdesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
	mdesc.Usage = D3D11_USAGE_DEFAULT;
	// Multisampling quality and count
	// The default sampler mode, with no anti-aliasing, has a count of 1 and a quality level of 0.
	mdesc.SampleDesc.Quality = 0;
	mdesc.SampleDesc.Count = 1;
	mdesc.MipLevels = 1;
	mdesc.ArraySize = 1;

 
	m_Device->CreateTexture2D(&mdesc, NULL, &pTexture);


	// QI IDXGIResource interface to synchronized shared surface.
	IDXGIResource* pDXGIResource = NULL;
	pTexture->QueryInterface(__uuidof(IDXGIResource), (LPVOID*)&pDXGIResource);
	HANDLE handle = NULL;
	// obtain handle to IDXGIResource object.
	pDXGIResource->GetSharedHandle(&handle);
	pDXGIResource->Release();


	ID3D11Resource * tempResource11;
	HRESULT openResult = m_Device->OpenSharedResource(handle, __uuidof(ID3D11Resource), (void**)(&tempResource11));
	m_Device->CreateShaderResourceView(tempResource11, NULL, &m_Texture);


}
void RenderAPI_D3D11::RenderDistrutionPass()
{
 
	ID3D11DeviceContext* m_deviceContext = NULL;
	m_Device->GetImmediateContext(&m_deviceContext);


	if (pcolorBuffer&&pTexture)
	{
		m_deviceContext->CopyResource(pTexture, pcolorBuffer);
		
	}
	unsigned int stride;
	unsigned int offset;

	 

	// Set vertex buffer stride and offset.
	stride = sizeof(VertexType);
	offset = 0;

	// Set the vertex buffer to active in the input assembler so it can be rendered.
	m_deviceContext->IASetVertexBuffers(0, 1, &m_vertexBuffer, &stride, &offset);

	// Set the index buffer to active in the input assembler so it can be rendered.
	m_deviceContext->IASetIndexBuffer(m_indexBuffer, DXGI_FORMAT_R32_UINT, 0);

	// Set the type of primitive that should be rendered from this vertex buffer, in this case triangles.
	m_deviceContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

 
	// Now set the constant buffer in the vertex shader with the updated values.
	m_deviceContext->VSSetConstantBuffers(bufferNumber, 1, &m_matrixBuffer);


	if (m_Texture)
	{
		// Set shader texture resource in the pixel shader.
		m_deviceContext->PSSetShaderResources(0, 1, &m_Texture);
	}



	// Set the vertex input layout.
	m_deviceContext->IASetInputLayout(m_layout);

	// Set the vertex and pixel shaders that will be used to render this triangle.
	m_deviceContext->VSSetShader(m_vertexShader, NULL, 0);
	m_deviceContext->PSSetShader(m_pixelShader, NULL, 0);

	// Set the sampler state in the pixel shader.
	m_deviceContext->PSSetSamplers(0, 1, &m_sampleState);

	// Render the triangle.
	m_deviceContext->DrawIndexed(m_indexCount, 0, 0);
 
	m_deviceContext->Release();
}

void RenderAPI_D3D11::ReleaseResources()
{
	SAFE_RELEASE(m_VB);
	SAFE_RELEASE(m_CB);
	SAFE_RELEASE(m_VertexShader);
	SAFE_RELEASE(m_PixelShader);
	SAFE_RELEASE(m_InputLayout);
	SAFE_RELEASE(m_RasterState);
	SAFE_RELEASE(m_BlendState);
	SAFE_RELEASE(m_DepthState);
}


void RenderAPI_D3D11::DrawSimpleTriangles(const float worldMatrix[16], int triangleCount, const void* verticesFloat3Byte4)
{
	ID3D11DeviceContext* ctx = NULL;
	m_Device->GetImmediateContext(&ctx);

	// Set basic render state
	ctx->OMSetDepthStencilState(m_DepthState, 0);
	ctx->RSSetState(m_RasterState);
	ctx->OMSetBlendState(m_BlendState, NULL, 0xFFFFFFFF);

	// Update constant buffer - just the world matrix in our case
	ctx->UpdateSubresource(m_CB, 0, NULL, worldMatrix, 64, 0);

	// Set shaders
	ctx->VSSetConstantBuffers(0, 1, &m_CB);
	ctx->VSSetShader(m_VertexShader, NULL, 0);
	ctx->PSSetShader(m_PixelShader, NULL, 0);

	// Update vertex buffer
	const int kVertexSize = 12 + 4;
	ctx->UpdateSubresource(m_VB, 0, NULL, verticesFloat3Byte4, triangleCount * 3 * kVertexSize, 0);

	// set input assembler data and draw
	ctx->IASetInputLayout(m_InputLayout);
	ctx->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
	UINT stride = kVertexSize;
	UINT offset = 0;
	ctx->IASetVertexBuffers(0, 1, &m_VB, &stride, &offset);
	ctx->Draw(triangleCount * 3, 0);

	ctx->Release();
}


void* RenderAPI_D3D11::BeginModifyTexture(void* textureHandle, int textureWidth, int textureHeight, int* outRowPitch)
{
	const int rowPitch = textureWidth * 4;
	// Just allocate a system memory buffer here for simplicity
	unsigned char* data = new unsigned char[rowPitch * textureHeight];
	*outRowPitch = rowPitch;
	return data;
}


void RenderAPI_D3D11::EndModifyTexture(void* textureHandle, int textureWidth, int textureHeight, int rowPitch, void* dataPtr)
{
	ID3D11Texture2D* d3dtex = (ID3D11Texture2D*)textureHandle;
	assert(d3dtex);

	ID3D11DeviceContext* ctx = NULL;
	m_Device->GetImmediateContext(&ctx);
	// Update texture data, and free the memory buffer
	ctx->UpdateSubresource(d3dtex, 0, NULL, dataPtr, rowPitch, 0);
	delete[] (unsigned char*)dataPtr;
	ctx->Release();
}


void* RenderAPI_D3D11::BeginModifyVertexBuffer(void* bufferHandle, size_t* outBufferSize)
{
	ID3D11Buffer* d3dbuf = (ID3D11Buffer*)bufferHandle;
	assert(d3dbuf);
	D3D11_BUFFER_DESC desc;
	d3dbuf->GetDesc(&desc);
	*outBufferSize = desc.ByteWidth;

	ID3D11DeviceContext* ctx = NULL;
	m_Device->GetImmediateContext(&ctx);
	D3D11_MAPPED_SUBRESOURCE mapped;
	ctx->Map(d3dbuf, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped);
	ctx->Release();

	return mapped.pData;
}


void RenderAPI_D3D11::EndModifyVertexBuffer(void* bufferHandle)
{
	ID3D11Buffer* d3dbuf = (ID3D11Buffer*)bufferHandle;
	assert(d3dbuf);

	ID3D11DeviceContext* ctx = NULL;
	m_Device->GetImmediateContext(&ctx);
	ctx->Unmap(d3dbuf, 0);
	ctx->Release();
}
 
void RenderAPI_D3D11::SetRenderTexture(void* colorBuffer, void* depthBuffer, int textureWidth, int textureHeight)
{
	pcolorBuffer =  (ID3D11Texture2D *)colorBuffer;
	pdepthBuffer = (ID3D11Texture2D *)depthBuffer;
 
}

#endif // #if SUPPORT_D3D11
