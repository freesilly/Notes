### ShaderLab结构
Properties
{

}
SubShader
{
	Pass
	 {
		CGPROGRAM/ HLSLPROGRAM
		
		#pragma vertex vert_name
		#pragma  fragment frag_name

		在Properties中出现的属性要再声明一次
		两个struct（a2v,v2f）
		两个函数分别是vert_name和frag_name

		ENDCG/ ENDHLSL
	}
}

![[Pasted image 20250904151804.png]]
