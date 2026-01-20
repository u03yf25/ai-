# python-
import os
import json
import random
import warnings
from io import BytesIO
from typing import List, Dict, Tuple

import openai
from PIL import Image, ImageDraw, ImageFont
from stability_sdk import client
import stability_sdk.interfaces.gooseai.generation.generation_pb2 as generation

class ComicPanel:
    """漫画分镜类"""
    def __init__(self, number: int, description: str, text: str):
        self.number = number
        self.description = description
        self.text = text
        self.image = None

class ComicGenerator:
    """漫画生成器类"""
    
    def __init__(self, openai_api_key: str, stability_api_key: str):
        # 初始化OpenAI客户端
        self.openai_client = openai.OpenAI(api_key=openai_api_key)
        
        # 初始化Stability AI客户端
        self.stability_api = client.StabilityInference(
            key=stability_api_key,
            verbose=True,
            engine="stable-diffusion-xl-1024-v1-0",
        )
        
        # 随机种子，确保生成的图像风格一致
        self.seed = random.randint(0, 1000000000)
        
        # 加载字体（可以替换为你喜欢的漫画字体）
        try:
            self.font = ImageFont.truetype("comic_font.ttf", 24)
        except:
            self.font = ImageFont.load_default()

    def generate_panels(self, story: str, num_panels: int = 6) -> List[ComicPanel]:
        """
        将故事文本生成为漫画分镜
        
        Args:
            story: 完整的故事文本
            num_panels: 生成的分镜数量
            
        Returns:
            漫画分镜列表
        """
        prompt = f"""
        将以下故事改编为{num_panels}格漫画的分镜描述：
        
        故事：{story}
        
        请按照以下格式输出，不要包含其他内容：
        [
            {{
                "number": 1,
                "description": "第一格的详细画面描述，包含角色、场景、动作",
                "text": "第一格的对话或旁白文字"
            }},
            ...
        ]
        """
        
        try:
            response = self.openai_client.chat.completions.create(
                model="gpt-3.5-turbo",
                messages=[
                    {"role": "system", "content": "你是一位专业的漫画分镜师，擅长将故事转化为生动的漫画分镜。"},
                    {"role": "user", "content": prompt}
                ],
                temperature=0.7,
                max_tokens=1000
            )
            
            panels_data = json.loads(response.choices[0].message.content)
            return [ComicPanel(**panel) for panel in panels_data]
            
        except Exception as e:
            print(f"生成漫画分镜时出错: {e}")
            return []

    def generate_panel_image(self, panel: ComicPanel, style: str = "漫画风格") -> Image.Image:
        """
        为单个分镜生成图像
        
        Args:
            panel: 漫画分镜对象
            style: 漫画风格描述
            
        Returns:
            生成的图像对象
        """
        prompt = f"{panel.description}, {style}, 线条清晰, 色彩鲜艳, 漫画分镜"
        
        try:
            answers = self.stability_api.generate(
                prompt=prompt,
                seed=self.seed,
                steps=30,
                cfg_scale=8.0,
                width=1024,
                height=1024,
                sampler=generation.SAMPLER_K_DPMPP_2M
            )
            
            for resp in answers:
                for artifact in resp.artifacts:
                    if artifact.finish_reason == generation.FILTER:
                        warnings.warn(
                            "您的请求触发了安全过滤，请修改提示词后重试。"
                        )
                        return None
                        
                    if artifact.type == generation.ARTIFACT_IMAGE:
                        return Image.open(BytesIO(artifact.binary))
                        
        except Exception as e:
            print(f"生成图像时出错: {e}")
            return None
            
        return None

    def add_text_to_panel(self, image: Image.Image, text: str) -> Image.Image:
        """
        为漫画分镜添加文字
        
        Args:
            image: 漫画图像
            text: 要添加的文字
            
        Returns:
            添加文字后的图像
        """
        # 创建绘图对象
        draw = ImageDraw.Draw(image)
        
        # 计算文字位置（底部居中）
        text_width, text_height = draw.textsize(text, font=self.font)
        x = (image.width - text_width) // 2
        y = image.height - text_height - 20
        
        # 添加文字背景
        bg_rectangle = [x-10, y-10, x+text_width+10, y+text_height+10]
        draw.rectangle(bg_rectangle, fill="white")
        
        # 添加文字
        draw.text((x, y), text, font=self.font, fill="black")
        
        return image

    def create_comic_strip(self, panels: List[ComicPanel]) -> Image.Image:
        """
        将多个分镜合成为完整的漫画条
        
        Args:
            panels: 漫画分镜列表
            
        Returns:
            合成的漫画条图像
        """
        if not panels:
            return None
            
        # 计算漫画条的尺寸
        panel_width = panels[0].image.width if panels[0].image else 1024
        panel_height = panels[0].image.height if panels[0].image else 1024
        
        # 创建漫画条画布（横向排列）
        strip_width = panel_width * len(panels)
        strip_height = panel_height
        strip_image = Image.new('RGB', (strip_width, strip_height), color='white')
        
        # 将每个分镜粘贴到漫画条上
        for i, panel in enumerate(panels):
            if panel.image:
                strip_image.paste(panel.image, (i * panel_width, 0))
                
        return strip_image

    def generate_comic(self, story: str, style: str = "漫画风格", num_panels: int = 6) -> Tuple[List[ComicPanel], Image.Image]:
        """
        完整的漫画生成流程
        
        Args:
            story: 故事文本
            style: 漫画风格
            num_panels: 分镜数量
            
        Returns:
            分镜列表和合成的漫画条
        """
        print("正在生成漫画分镜...")
        panels = self.generate_panels(story, num_panels)
        
        if not panels:
            return [], None
            
        print(f"正在生成{len(panels)}格漫画图像...")
        for i, panel in enumerate(panels):
            print(f"正在生成第{i+1}格...")
            image = self.generate_panel_image(panel, style)
            
            if image:
                # 添加文字到图像
                panel.image = self.add_text_to_panel(image, panel.text)
        
        print("正在合成漫画条...")
        comic_strip = self.create_comic_strip(panels)
        
        return panels, comic_strip
