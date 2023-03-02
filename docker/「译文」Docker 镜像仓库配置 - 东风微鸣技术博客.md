> 👉️**URL:** [https://docs.docker.com/registry/configuration/](https://docs.docker.com/registry/configuration/)
> 
> 📝**Description:**
> 
> Explains how to configure a registry

注册表的配置是基于 YAML 文件的，详见下文。虽然它带有合理的默认值，但在将你的系统转移到生产中之前，你应该详尽地审查它。

## RHEL/CentOS 7 的配置[](https://ewhisper.cn/posts/48670/#RHEL-CentOS-7-%20%E7%9A%84%E9%85%8D%E7%BD%AE)

> 📚 **Quote:**
> 
> [How to secure private docker registry on Red Hat Enterprise Linux 7 ? - Red Hat Customer Portal](https://access.redhat.com/solutions/3175391)

`/etc/docker-distribution/registry/config.yml`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br></pre></td><td><pre><code><span>version:</span> <span>0.1</span><br><span>log:</span><br>  <span>fields:</span><br>    <span>service:</span> <span>registry</span><br><span>storage:</span><br>    <span>cache:</span><br>        <span>layerinfo:</span> <span>inmemory</span><br>    <span>filesystem:</span><br>        <span>rootdirectory:</span> <span>/data/registry</span><br>    <span>maintenance:</span><br>        <span>uploadpurging:</span><br>            <span>enabled:</span> <span>true</span><br>            <span>age:</span> <span>1680h</span><br>            <span>interval:</span> <span>24h</span><br>            <span>dryrun:</span> <span>false</span><br>    <span>delete:</span><br>        <span>enabled:</span> <span>true</span><br><span>http:</span><br>    <span>addr:</span> <span>:5000</span></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

> ℹ️ **Info:**
> 
> uploadpurging 2 个月之前的；
> 
> 可以删除镜像 blobs 和 manifests by digest

## Debian 的配置[](https://ewhisper.cn/posts/48670/#Debian-%20%E7%9A%84%E9%85%8D%E7%BD%AE)

`/etc/docker/registry/config.yml`:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br></pre></td><td><pre><code><span>version:</span> <span>0.1</span><br><span>log:</span><br>  <span>fields:</span><br>    <span>service:</span> <span>registry</span><br><span>storage:</span><br>  <span>cache:</span><br>    <span>blobdescriptor:</span> <span>inmemory</span><br>  <span>filesystem:</span><br>    <span>rootdirectory:</span> <span>/var/lib/docker-registry</span><br>  <span>maintenance:</span><br>    <span>uploadpurging:</span><br>      <span>enabled:</span> <span>true</span><br>      <span>age:</span> <span>744h</span><br>      <span>interval:</span> <span>24h</span><br>      <span>dryrun:</span> <span>false</span><br>  <span>delete:</span><br>    <span>enabled:</span> <span>true</span><br><span>http:</span><br>  <span>addr:</span> <span>:5000</span><br>  <span>headers:</span><br>    <span>X-Content-Type-Options:</span> [<span>nosniff</span>]<br><span>health:</span><br>  <span>storagedriver:</span><br>    <span>enabled:</span> <span>true</span><br>    <span>interval:</span> <span>10s</span><br>    <span>threshold:</span> <span>3</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

> ℹ️ **Info:**
> 
> uploadpurging 1 个月之前的；
> 
> 可以删除镜像 blobs 和 manifests by digest

## 覆盖特定配置选项[](https://ewhisper.cn/posts/48670/#%E8%A6%86%E7%9B%96%E7%89%B9%E5%AE%9A%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9)

在一个典型的设置中，你从官方镜像运行注册表，你可以通过传递 `-e` 参数到你的 `docker run` 或使用`ENV` 指令从 Dockerfile 中指定一个配置变量。

要覆盖配置选项，请创建一个名为 `REGISTRY_variable` 的环境变量，其中 `variable` 是配置选项的名称，`_`（下划线）表示缩进级别。例如，你可以配置 `filesystem` 存储后端的`rootdirectory`:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span>storage:</span><br>  <span>filesystem:</span><br>    <span>rootdirectory:</span> <span>/var/lib/registry</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

要覆盖这个值，可以这样设置一个环境变量：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/somewhere</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

> 注意：创建一个带有环境变量的基本配置文件，可以通过配置来调整各个数值。不建议用环境变量覆盖配置部分。

## 覆盖整个配置文件[](https://ewhisper.cn/posts/48670/#%E8%A6%86%E7%9B%96%E6%95%B4%E4%B8%AA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

如果默认配置对您的使用来说不是一个良好的基础，或者如果您在覆盖环境中的键时遇到了问题，您可以通过将其作为一个卷挂载到容器中来指定一个备用 YAML 配置文件。

通常，从头创建一个新的配置文件，命名为 `config.yaml`, 然后在`docker run` 命令中指定它：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>$ docker run -d -p 5000:5000 --restart=always --name registry \<br>             -v `<span>pwd</span>`/config.yml:/etc/docker/registry/config.yml \<br>             registry:2<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

使用这个示例 YAML 文件作为起点：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br></pre></td><td><pre><code><span>version:</span> <span>0.1</span><br><span>log:</span><br>  <span>fields:</span><br>    <span>service:</span> <span>registry</span><br><span>storage:</span><br>  <span>cache:</span><br>    <span>blobdescriptor:</span> <span>inmemory</span><br>  <span>filesystem:</span><br>    <span>rootdirectory:</span> <span>/var/lib/registry</span><br><span>http:</span><br>  <span>addr:</span> <span>:5000</span><br>  <span>headers:</span><br>    <span>X-Content-Type-Options:</span> [<span>nosniff</span>]<br><span>auth:</span><br>  <span>htpasswd:</span><br>    <span>realm:</span> <span>basic-realm</span><br>    <span>path:</span> <span>/etc/registry</span><br><span>health:</span><br>  <span>storagedriver:</span><br>    <span>enabled:</span> <span>true</span><br>    <span>interval:</span> <span>10s</span><br>    <span>threshold:</span> <span>3</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

## 配置选项列表[](https://ewhisper.cn/posts/48670/#%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9%E5%88%97%E8%A1%A8)

这些都是注册表的配置选项。列表中的某些选项是互斥的。在完成配置之前，请阅读有关每个选项的详细参考信息。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br><span>77</span><br><span>78</span><br><span>79</span><br><span>80</span><br><span>81</span><br><span>82</span><br><span>83</span><br><span>84</span><br><span>85</span><br><span>86</span><br><span>87</span><br><span>88</span><br><span>89</span><br><span>90</span><br><span>91</span><br><span>92</span><br><span>93</span><br><span>94</span><br><span>95</span><br><span>96</span><br><span>97</span><br><span>98</span><br><span>99</span><br><span>100</span><br><span>101</span><br><span>102</span><br><span>103</span><br><span>104</span><br><span>105</span><br><span>106</span><br><span>107</span><br><span>108</span><br><span>109</span><br><span>110</span><br><span>111</span><br><span>112</span><br><span>113</span><br><span>114</span><br><span>115</span><br><span>116</span><br><span>117</span><br><span>118</span><br><span>119</span><br><span>120</span><br><span>121</span><br><span>122</span><br><span>123</span><br><span>124</span><br><span>125</span><br><span>126</span><br><span>127</span><br><span>128</span><br><span>129</span><br><span>130</span><br><span>131</span><br><span>132</span><br><span>133</span><br><span>134</span><br><span>135</span><br><span>136</span><br><span>137</span><br><span>138</span><br><span>139</span><br><span>140</span><br><span>141</span><br><span>142</span><br><span>143</span><br><span>144</span><br><span>145</span><br><span>146</span><br><span>147</span><br><span>148</span><br><span>149</span><br><span>150</span><br><span>151</span><br><span>152</span><br><span>153</span><br><span>154</span><br><span>155</span><br><span>156</span><br><span>157</span><br><span>158</span><br><span>159</span><br><span>160</span><br><span>161</span><br><span>162</span><br><span>163</span><br><span>164</span><br><span>165</span><br><span>166</span><br><span>167</span><br><span>168</span><br><span>169</span><br><span>170</span><br><span>171</span><br><span>172</span><br><span>173</span><br><span>174</span><br><span>175</span><br><span>176</span><br><span>177</span><br><span>178</span><br><span>179</span><br><span>180</span><br><span>181</span><br><span>182</span><br><span>183</span><br><span>184</span><br><span>185</span><br><span>186</span><br><span>187</span><br><span>188</span><br><span>189</span><br><span>190</span><br><span>191</span><br><span>192</span><br><span>193</span><br><span>194</span><br><span>195</span><br><span>196</span><br><span>197</span><br><span>198</span><br><span>199</span><br><span>200</span><br><span>201</span><br><span>202</span><br><span>203</span><br><span>204</span><br><span>205</span><br><span>206</span><br><span>207</span><br><span>208</span><br><span>209</span><br><span>210</span><br><span>211</span><br><span>212</span><br><span>213</span><br><span>214</span><br><span>215</span><br><span>216</span><br><span>217</span><br><span>218</span><br><span>219</span><br><span>220</span><br><span>221</span><br><span>222</span><br><span>223</span><br><span>224</span><br><span>225</span><br><span>226</span><br><span>227</span><br><span>228</span><br><span>229</span><br><span>230</span><br><span>231</span><br><span>232</span><br><span>233</span><br><span>234</span><br><span>235</span><br><span>236</span><br><span>237</span><br><span>238</span><br></pre></td><td><pre><code><span>version:</span> <span>0.1</span><br><span>log:</span><br>  <span>accesslog:</span><br>    <span>disabled:</span> <span>true</span><br>  <span>level:</span> <span>debug</span><br>  <span>formatter:</span> <span>text</span><br>  <span>fields:</span><br>    <span>service:</span> <span>registry</span><br>    <span>environment:</span> <span>staging</span><br>  <span>hooks:</span><br>    <span>-</span> <span>type:</span> <span>mail</span><br>      <span>disabled:</span> <span>true</span><br>      <span>levels:</span><br>        <span>-</span> <span>panic</span><br>      <span>options:</span><br>        <span>smtp:</span><br>          <span>addr:</span> <span>mail.example.com:25</span><br>          <span>username:</span> <span>mailuser</span><br>          <span>password:</span> <span>password</span><br>          <span>insecure:</span> <span>true</span><br>        <span>from:</span> <span>sender@example.com</span><br>        <span>to:</span><br>          <span>-</span> <span>errors@example.com</span><br><span>loglevel:</span> <span>debug</span> <span># deprecated: use "log"</span><br><span>storage:</span><br>  <span>filesystem:</span><br>    <span>rootdirectory:</span> <span>/var/lib/registry</span><br>    <span>maxthreads:</span> <span>100</span><br>  <span>azure:</span><br>    <span>accountname:</span> <span>accountname</span><br>    <span>accountkey:</span> <span>base64encodedaccountkey</span><br>    <span>container:</span> <span>containername</span><br>  <span>gcs:</span><br>    <span>bucket:</span> <span>bucketname</span><br>    <span>keyfile:</span> <span>/path/to/keyfile</span><br>    <span>credentials:</span><br>      <span>type:</span> <span>service_account</span><br>      <span>project_id:</span> <span>project_id_string</span><br>      <span>private_key_id:</span> <span>private_key_id_string</span><br>      <span>private_key:</span> <span>private_key_string</span><br>      <span>client_email:</span> <span>client@example.com</span><br>      <span>client_id:</span> <span>client_id_string</span><br>      <span>auth_uri:</span> <span>http://example.com/auth_uri</span><br>      <span>token_uri:</span> <span>http://example.com/token_uri</span><br>      <span>auth_provider_x509_cert_url:</span> <span>http://example.com/provider_cert_url</span><br>      <span>client_x509_cert_url:</span> <span>http://example.com/client_cert_url</span><br>    <span>rootdirectory:</span> <span>/gcs/object/name/prefix</span><br>    <span>chunksize:</span> <span>5242880</span><br>  <span>s3:</span><br>    <span>accesskey:</span> <span>awsaccesskey</span><br>    <span>secretkey:</span> <span>awssecretkey</span><br>    <span>region:</span> <span>us-west-1</span><br>    <span>regionendpoint:</span> <span>http://myobjects.local</span><br>    <span>bucket:</span> <span>bucketname</span><br>    <span>encrypt:</span> <span>true</span><br>    <span>keyid:</span> <span>mykeyid</span><br>    <span>secure:</span> <span>true</span><br>    <span>v4auth:</span> <span>true</span><br>    <span>chunksize:</span> <span>5242880</span><br>    <span>multipartcopychunksize:</span> <span>33554432</span><br>    <span>multipartcopymaxconcurrency:</span> <span>100</span><br>    <span>multipartcopythresholdsize:</span> <span>33554432</span><br>    <span>rootdirectory:</span> <span>/s3/object/name/prefix</span><br>  <span>swift:</span><br>    <span>username:</span> <span>username</span><br>    <span>password:</span> <span>password</span><br>    <span>authurl:</span> <span>https://storage.myprovider.com/auth/v1.0</span> <span>or</span> <span>https://storage.myprovider.com/v2.0</span> <span>or</span> <span>https://storage.myprovider.com/v3/auth</span><br>    <span>tenant:</span> <span>tenantname</span><br>    <span>tenantid:</span> <span>tenantid</span><br>    <span>domain:</span> <span>domain</span> <span>name</span> <span>for</span> <span>Openstack</span> <span>Identity</span> <span>v3</span> <span>API</span><br>    <span>domainid:</span> <span>domain</span> <span>id</span> <span>for</span> <span>Openstack</span> <span>Identity</span> <span>v3</span> <span>API</span><br>    <span>insecureskipverify:</span> <span>true</span><br>    <span>region:</span> <span>fr</span><br>    <span>container:</span> <span>containername</span><br>    <span>rootdirectory:</span> <span>/swift/object/name/prefix</span><br>  <span>oss:</span><br>    <span>accesskeyid:</span> <span>accesskeyid</span><br>    <span>accesskeysecret:</span> <span>accesskeysecret</span><br>    <span>region:</span> <span>OSS</span> <span>region</span> <span>name</span><br>    <span>endpoint:</span> <span>optional</span> <span>endpoints</span><br>    <span>internal:</span> <span>optional</span> <span>internal</span> <span>endpoint</span><br>    <span>bucket:</span> <span>OSS</span> <span>bucket</span><br>    <span>encrypt:</span> <span>optional</span> <span>data</span> <span>encryption</span> <span>setting</span><br>    <span>secure:</span> <span>optional</span> <span>ssl</span> <span>setting</span><br>    <span>chunksize:</span> <span>optional</span> <span>size</span> <span>valye</span><br>    <span>rootdirectory:</span> <span>optional</span> <span>root</span> <span>directory</span><br>  <span>inmemory:</span>  <span># This driver takes no parameters</span><br>  <span>delete:</span><br>    <span>enabled:</span> <span>false</span><br>  <span>redirect:</span><br>    <span>disable:</span> <span>false</span><br>  <span>cache:</span><br>    <span>blobdescriptor:</span> <span>redis</span><br>  <span>maintenance:</span><br>    <span>uploadpurging:</span><br>      <span>enabled:</span> <span>true</span><br>      <span>age:</span> <span>168h</span><br>      <span>interval:</span> <span>24h</span><br>      <span>dryrun:</span> <span>false</span><br>    <span>readonly:</span><br>      <span>enabled:</span> <span>false</span><br><span>auth:</span><br>  <span>silly:</span><br>    <span>realm:</span> <span>silly-realm</span><br>    <span>service:</span> <span>silly-service</span><br>  <span>token:</span><br>    <span>autoredirect:</span> <span>true</span><br>    <span>realm:</span> <span>token-realm</span><br>    <span>service:</span> <span>token-service</span><br>    <span>issuer:</span> <span>registry-token-issuer</span><br>    <span>rootcertbundle:</span> <span>/root/certs/bundle</span><br>  <span>htpasswd:</span><br>    <span>realm:</span> <span>basic-realm</span><br>    <span>path:</span> <span>/path/to/htpasswd</span><br><span>middleware:</span><br>  <span>registry:</span><br>    <span>-</span> <span>name:</span> <span>ARegistryMiddleware</span><br>      <span>options:</span><br>        <span>foo:</span> <span>bar</span><br>  <span>repository:</span><br>    <span>-</span> <span>name:</span> <span>ARepositoryMiddleware</span><br>      <span>options:</span><br>        <span>foo:</span> <span>bar</span><br>  <span>storage:</span><br>    <span>-</span> <span>name:</span> <span>cloudfront</span><br>      <span>options:</span><br>        <span>baseurl:</span> <span>https://my.cloudfronted.domain.com/</span><br>        <span>privatekey:</span> <span>/path/to/pem</span><br>        <span>keypairid:</span> <span>cloudfrontkeypairid</span><br>        <span>duration:</span> <span>3000s</span><br>        <span>ipfilteredby:</span> <span>awsregion</span><br>        <span>awsregion:</span> <span>us-east-1,</span> <span>use-east-2</span><br>        <span>updatefrenquency:</span> <span>12h</span><br>        <span>iprangesurl:</span> <span>https://ip-ranges.amazonaws.com/ip-ranges.json</span><br>  <span>storage:</span><br>    <span>-</span> <span>name:</span> <span>redirect</span><br>      <span>options:</span><br>        <span>baseurl:</span> <span>https://example.com/</span><br><span>reporting:</span><br>  <span>bugsnag:</span><br>    <span>apikey:</span> <span>bugsnagapikey</span><br>    <span>releasestage:</span> <span>bugsnagreleasestage</span><br>    <span>endpoint:</span> <span>bugsnagendpoint</span><br>  <span>newrelic:</span><br>    <span>licensekey:</span> <span>newreliclicensekey</span><br>    <span>name:</span> <span>newrelicname</span><br>    <span>verbose:</span> <span>true</span><br><span>http:</span><br>  <span>addr:</span> <span>localhost:5000</span><br>  <span>prefix:</span> <span>/my/nested/registry/</span><br>  <span>host:</span> <span>https://myregistryaddress.org:5000</span><br>  <span>secret:</span> <span>asecretforlocaldevelopment</span><br>  <span>relativeurls:</span> <span>false</span><br>  <span>draintimeout:</span> <span>60s</span><br>  <span>tls:</span><br>    <span>certificate:</span> <span>/path/to/x509/public</span><br>    <span>key:</span> <span>/path/to/x509/private</span><br>    <span>clientcas:</span><br>      <span>-</span> <span>/path/to/ca.pem</span><br>      <span>-</span> <span>/path/to/another/ca.pem</span><br>    <span>letsencrypt:</span><br>      <span>cachefile:</span> <span>/path/to/cache-file</span><br>      <span>email:</span> <span>emailused@letsencrypt.com</span><br>      <span>hosts:</span> [<span>myregistryaddress.org</span>]<br>  <span>debug:</span><br>    <span>addr:</span> <span>localhost:5001</span><br>    <span>prometheus:</span><br>      <span>enabled:</span> <span>true</span><br>      <span>path:</span> <span>/metrics</span><br>  <span>headers:</span><br>    <span>X-Content-Type-Options:</span> [<span>nosniff</span>]<br>  <span>http2:</span><br>    <span>disabled:</span> <span>false</span><br><span>notifications:</span><br>  <span>events:</span><br>    <span>includereferences:</span> <span>true</span><br>  <span>endpoints:</span><br>    <span>-</span> <span>name:</span> <span>alistener</span><br>      <span>disabled:</span> <span>false</span><br>      <span>url:</span> <span>https://my.listener.com/event</span><br>      <span>headers:</span> <span>&lt;http.Header&gt;</span><br>      <span>timeout:</span> <span>1s</span><br>      <span>threshold:</span> <span>10</span><br>      <span>backoff:</span> <span>1s</span><br>      <span>ignoredmediatypes:</span><br>        <span>-</span> <span>application/octet-stream</span><br>      <span>ignore:</span><br>        <span>mediatypes:</span><br>           <span>-</span> <span>application/octet-stream</span><br>        <span>actions:</span><br>           <span>-</span> <span>pull</span><br><span>redis:</span><br>  <span>addr:</span> <span>localhost:6379</span><br>  <span>password:</span> <span>asecret</span><br>  <span>db:</span> <span>0</span><br>  <span>dialtimeout:</span> <span>10ms</span><br>  <span>readtimeout:</span> <span>10ms</span><br>  <span>writetimeout:</span> <span>10ms</span><br>  <span>pool:</span><br>    <span>maxidle:</span> <span>16</span><br>    <span>maxactive:</span> <span>64</span><br>    <span>idletimeout:</span> <span>300s</span><br><span>health:</span><br>  <span>storagedriver:</span><br>    <span>enabled:</span> <span>true</span><br>    <span>interval:</span> <span>10s</span><br>    <span>threshold:</span> <span>3</span><br>  <span>file:</span><br>    <span>-</span> <span>file:</span> <span>/path/to/checked/file</span><br>      <span>interval:</span> <span>10s</span><br>  <span>http:</span><br>    <span>-</span> <span>uri:</span> <span>http://server.to.check/must/return/200</span><br>      <span>headers:</span><br>        <span>Authorization:</span> [<span>Basic</span> <span>QWxhZGRpbjpvcGVuIHNlc2FtZQ==</span>]<br>      <span>statuscode:</span> <span>200</span><br>      <span>timeout:</span> <span>3s</span><br>      <span>interval:</span> <span>10s</span><br>      <span>threshold:</span> <span>3</span><br>  <span>tcp:</span><br>    <span>-</span> <span>addr:</span> <span>redis-server.domain.com:6379</span><br>      <span>timeout:</span> <span>3s</span><br>      <span>interval:</span> <span>10s</span><br>      <span>threshold:</span> <span>3</span><br><span>proxy:</span><br>  <span>remoteurl:</span> <span>https://registry-1.docker.io</span><br>  <span>username:</span> [<span>username</span>]<br>  <span>password:</span> [<span>password</span>]<br><span>compatibility:</span><br>  <span>schema1:</span><br>    <span>signingkeyfile:</span> <span>/etc/registry/key.json</span><br>    <span>enabled:</span> <span>true</span><br><span>validation:</span><br>  <span>manifests:</span><br>    <span>urls:</span><br>      <span>allow:</span><br>        <span>-</span> <span>^https?://([^/]+\.)*example\.com/</span><br>      <span>deny:</span><br>        <span>-</span> <span>^https?://www\.example\.com/</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

在某些情况下，配置选项是 **可选** 的，但它包含标记为 **必需** 的子选项。在这些情况下，可以省略父节点及其所有子节点。但是，如果包含父元素，则还必须包含所有标记为 **required** 的子元素。

## `version`[](https://ewhisper.cn/posts/48670/#version)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>version:</span> <span>0.1</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`version`选项是必需的。它指定配置的版本。预计它将保留顶级字段，以便在解析配置文件的其余部分之前进行一致的版本检查。

## `log`[](https://ewhisper.cn/posts/48670/#log)

`log`小节配置日志系统的行为。日志系统将所有内容输出到 stdout。您可以使用此配置部分调整粒度和格式。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code><span>log:</span><br>  <span>accesslog:</span><br>    <span>disabled:</span> <span>true</span><br>  <span>level:</span> <span>debug</span><br>  <span>formatter:</span> <span>text</span><br>  <span>fields:</span><br>    <span>service:</span> <span>registry</span><br>    <span>environment:</span> <span>staging</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

| 参数 | Required | 描述 |
| --- | --- | --- |
| `level` | no | 设置日志输出的灵敏度。允许的值是 `error`、`warn`、`info` 和`debug`。默认值是`info`。 |
| `formatter` | no | 这将选择日志输出的格式。格式主要影响日志行键控属性的编码方式。 可选项为 `text`, `json`, 和`logstash`. 默认 `text`. |
| `fields` | no | 字段名到值的映射。这些被添加到上下文的每个日志行中。这对于标识在其他系统中混杂的日志消息源非常有用。 |

### `accesslog`[](https://ewhisper.cn/posts/48670/#accesslog)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>accesslog:</span><br>  <span>disabled:</span> <span>true</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

在 `log` 中，`accesslog`配置访问日志系统的行为。缺省情况下，访问日志系统以组合日志格式 ([Apache Combined Log Format](https://httpd.apache.org/docs/2.4/logs.html#combined)) 输出到标准输出。可以通过将 boolean 标志 `disabled` 设置为 `true` 来禁用访问日志记录。

## `hooks`[](https://ewhisper.cn/posts/48670/#hooks)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><code><span>hooks:</span><br>  <span>-</span> <span>type:</span> <span>mail</span><br>    <span>levels:</span><br>      <span>-</span> <span>panic</span><br>    <span>options:</span><br>      <span>smtp:</span><br>        <span>addr:</span> <span>smtp.sendhost.com:25</span><br>        <span>username:</span> <span>sendername</span><br>        <span>password:</span> <span>password</span><br>        <span>insecure:</span> <span>true</span><br>      <span>from:</span> <span>name@sendhost.com</span><br>      <span>to:</span><br>        <span>-</span> <span>name@receivehost.com</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`hooks`小节配置 **日志** 钩子的行为。例如，本小节包括一个序列处理程序，您可以使用它来发送邮件。请参考 `log.level` 配置打印消息的级别。

## `loglevel`[](https://ewhisper.cn/posts/48670/#loglevel)

> **已弃用** : 请使用`log` 代替。

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>loglevel:</span> <span>debug</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

允许的值`error`, `warn`, `info` 和`debug`. 默认值 `info`.

## `storage`[](https://ewhisper.cn/posts/48670/#storage)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br></pre></td><td><pre><code><span>storage:</span><br>  <span>filesystem:</span><br>    <span>rootdirectory:</span> <span>/var/lib/registry</span><br>  <span>azure:</span><br>    <span>accountname:</span> <span>accountname</span><br>    <span>accountkey:</span> <span>base64encodedaccountkey</span><br>    <span>container:</span> <span>containername</span><br>  <span>gcs:</span><br>    <span>bucket:</span> <span>bucketname</span><br>    <span>keyfile:</span> <span>/path/to/keyfile</span><br>    <span>credentials:</span><br>      <span>type:</span> <span>service_account</span><br>      <span>project_id:</span> <span>project_id_string</span><br>      <span>private_key_id:</span> <span>private_key_id_string</span><br>      <span>private_key:</span> <span>private_key_string</span><br>      <span>client_email:</span> <span>client@example.com</span><br>      <span>client_id:</span> <span>client_id_string</span><br>      <span>auth_uri:</span> <span>http://example.com/auth_uri</span><br>      <span>token_uri:</span> <span>http://example.com/token_uri</span><br>      <span>auth_provider_x509_cert_url:</span> <span>http://example.com/provider_cert_url</span><br>      <span>client_x509_cert_url:</span> <span>http://example.com/client_cert_url</span><br>    <span>rootdirectory:</span> <span>/gcs/object/name/prefix</span><br>  <span>s3:</span><br>    <span>accesskey:</span> <span>awsaccesskey</span><br>    <span>secretkey:</span> <span>awssecretkey</span><br>    <span>region:</span> <span>us-west-1</span><br>    <span>regionendpoint:</span> <span>http://myobjects.local</span><br>    <span>bucket:</span> <span>bucketname</span><br>    <span>encrypt:</span> <span>true</span><br>    <span>keyid:</span> <span>mykeyid</span><br>    <span>secure:</span> <span>true</span><br>    <span>v4auth:</span> <span>true</span><br>    <span>chunksize:</span> <span>5242880</span><br>    <span>multipartcopychunksize:</span> <span>33554432</span><br>    <span>multipartcopymaxconcurrency:</span> <span>100</span><br>    <span>multipartcopythresholdsize:</span> <span>33554432</span><br>    <span>rootdirectory:</span> <span>/s3/object/name/prefix</span><br>  <span>swift:</span><br>    <span>username:</span> <span>username</span><br>    <span>password:</span> <span>password</span><br>    <span>authurl:</span> <span>https://storage.myprovider.com/auth/v1.0</span> <span>or</span> <span>https://storage.myprovider.com/v2.0</span> <span>or</span> <span>https://storage.myprovider.com/v3/auth</span><br>    <span>tenant:</span> <span>tenantname</span><br>    <span>tenantid:</span> <span>tenantid</span><br>    <span>domain:</span> <span>domain</span> <span>name</span> <span>for</span> <span>Openstack</span> <span>Identity</span> <span>v3</span> <span>API</span><br>    <span>domainid:</span> <span>domain</span> <span>id</span> <span>for</span> <span>Openstack</span> <span>Identity</span> <span>v3</span> <span>API</span><br>    <span>insecureskipverify:</span> <span>true</span><br>    <span>region:</span> <span>fr</span><br>    <span>container:</span> <span>containername</span><br>    <span>rootdirectory:</span> <span>/swift/object/name/prefix</span><br>  <span>oss:</span><br>    <span>accesskeyid:</span> <span>accesskeyid</span><br>    <span>accesskeysecret:</span> <span>accesskeysecret</span><br>    <span>region:</span> <span>OSS</span> <span>region</span> <span>name</span><br>    <span>endpoint:</span> <span>optional</span> <span>endpoints</span><br>    <span>internal:</span> <span>optional</span> <span>internal</span> <span>endpoint</span><br>    <span>bucket:</span> <span>OSS</span> <span>bucket</span><br>    <span>encrypt:</span> <span>optional</span> <span>data</span> <span>encryption</span> <span>setting</span><br>    <span>secure:</span> <span>optional</span> <span>ssl</span> <span>setting</span><br>    <span>chunksize:</span> <span>optional</span> <span>size</span> <span>valye</span><br>    <span>rootdirectory:</span> <span>optional</span> <span>root</span> <span>directory</span><br>  <span>inmemory:</span><br>  <span>delete:</span><br>    <span>enabled:</span> <span>false</span><br>  <span>cache:</span><br>    <span>blobdescriptor:</span> <span>inmemory</span><br>  <span>maintenance:</span><br>    <span>uploadpurging:</span><br>      <span>enabled:</span> <span>true</span><br>      <span>age:</span> <span>168h</span><br>      <span>interval:</span> <span>24h</span><br>      <span>dryrun:</span> <span>false</span><br>    <span>readonly:</span><br>      <span>enabled:</span> <span>false</span><br>  <span>redirect:</span><br>    <span>disable:</span> <span>false</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`storage`选项是 **必需** 的，它定义了使用哪个存储后端。您必须恰好配置一个后端。如果配置更多，注册表将返回一个错误。你可以选择这些后端存储驱动程序中的任何一个：

| 存储驱动 | 描述 |
| --- | --- |
| `filesystem` | 使用本地磁盘存储注册表文件。它是理想的开发，可能适合一些小规模的生产应用程序。See the [driver’s reference documentation](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/filesystem.md). |
| `azure` | Microsoft Azure Blob Storage. See the [driver’s reference documentation](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/azure.md). |
| `gcs` | Google Cloud Storage. See the [driver’s reference documentation](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/gcs.md). |
| `s3` | Amazon Simple Storage Service (S3) and 兼容对象 Storage Services. See the [driver’s reference documentation](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/s3.md). |
| `swift` | Openstack Swift object storage. See the [driver’s reference documentation](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/swift.md). |
| `oss` | Aliyun OSS for object storage. See the [driver’s reference documentation](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/oss.md). |

仅用于测试，您可以使用 `inmemory` [存储驱动程序](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/inmemory.md)。如果希望从易失性内存运行注册表，请使用 ram 磁盘上的 [`filesystem` driver](https://github.com/docker/docker.github.io/tree/master/registry/storage-drivers/filesystem.md)。

如果在 Windows 上部署注册表，则不建议从主机上挂载 Windows 卷。相反，您可以使用 S3 或 Azure 的后备数据存储。如果你使用一个 Windows 卷，到挂载点的 `PATH` 长度必须在 `MAX_PATH` 限制范围内（通常是 255 个字符），否则会出现这个错误：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>mkdir /XXX protocol<span> error</span> <span>and</span> your registry will<span> not</span> <span>function</span> properly.<br></code><p><i></i>XQUERY</p></pre></td></tr></tbody></table>

### `maintenance`[](https://ewhisper.cn/posts/48670/#maintenance)

目前仅有上传清除和只读 `maintenance` 两种方式。

### `uploadpurging`[](https://ewhisper.cn/posts/48670/#uploadpurging)

上传清除是一个后台过程，它定期从注册表的上传目录中删除孤立的文件。**缺省情况下启用上传清除功能**。设置上传目录清除功能，需要设置以下参数。

| 参数 | Required | 描述 |
| --- | --- | --- |
| `enabled` | yes | Set to `true` to enable upload purging. Defaults to `true`. |
| `age` | yes | 超过这个年龄的上传目录将被删除。默认`168h`(1 周）。 |
| `interval` | yes | 上传目录清理的时间间隔。Defaults to `24h`. |
| `dryrun` | yes | 将 `dryrun` 设置为 `true` 以获得将删除哪些目录的摘要。默认值为`false`。 |

> 注意：年龄和间隔是字符串，包含一个数字，可选的分数和一个单位后缀。例如：45m, 2h10m, 168h。

### `readonly`[](https://ewhisper.cn/posts/48670/#readonly)

如果 `maintenance` 下的 `readonly` 部分启用设置为 `true`，则不允许客户端写入注册表。这种模式对于临时阻止对后端存储的写入非常有用，这样就可以运行垃圾收集。在运行垃圾收集之前，应该重新启动注册表，将`readonly` 的`enabled`设置为 `true`。垃圾收集完成后，注册表可能会再次重启，这一次`readonly` 从配置中删除（或设置为`false`)。

### `delete`[](https://ewhisper.cn/posts/48670/#delete)

使用 `delete` 设置为 `enable` 可以删除镜像 blobs 和 manifests by digest。默认为`false`，但可以通过在配置文件中写入以下内容来启用：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>delete:</span><br>  <span>enabled:</span> <span>true</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

### `cache`[](https://ewhisper.cn/posts/48670/#cache)

使用 `cache` 结构可以启用缓存存储后端访问的数据。目前，这唯一可用的缓存字段提供了对 layer 元数据的快速访问，如果配置了，它将使用 `blobdescriptor` 字段。

你可以设置`blobdescriptor` 为 `redis` 或 `inmemory`. 如果设置为 `redis`, Redis 池缓存层元数据。如果设置为`inmemory`，内存映射缓存层元数据。

> 注意：以前，`blobdescriptor`被称为 `layerinfo`。虽然它们是等价的，但是`layerinfo` 已被弃用。

### `redirect`[](https://ewhisper.cn/posts/48670/#redirect)

`redirect`小节提供了用于管理内容后端重定向的配置。对于支持重定向的后端，默认情况下启用重定向。在某些部署场景中，您可能决定通过 Registry 路由所有数据，而不是重定向到后端。当使用不位于同一位置的后端或当注册表实例正在积极缓存时，这可能会更有效。

要禁用重定向，添加一个单独的标志 `disable`，在`redirect` 部分设置为`true`:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>redirect:</span><br>  <span>disable:</span> <span>true</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

## `auth`[](https://ewhisper.cn/posts/48670/#auth)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><code><span>auth:</span><br>  <span>silly:</span><br>    <span>realm:</span> <span>silly-realm</span><br>    <span>service:</span> <span>silly-service</span><br>  <span>token:</span><br>    <span>realm:</span> <span>token-realm</span><br>    <span>service:</span> <span>token-service</span><br>    <span>issuer:</span> <span>registry-token-issuer</span><br>    <span>rootcertbundle:</span> <span>/root/certs/bundle</span><br>  <span>htpasswd:</span><br>    <span>realm:</span> <span>basic-realm</span><br>    <span>path:</span> <span>/path/to/htpasswd</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`auth`选项是可选的。可能的认证提供者包括：

-   [`silly`](https://docs.docker.com/registry/configuration/#silly)
-   [`token`](https://docs.docker.com/registry/configuration/#token)
-   [`htpasswd`](https://docs.docker.com/registry/configuration/#htpasswd)
-   \[`none`\]

只能配置一个身份验证提供程序。

### `silly`[](https://ewhisper.cn/posts/48670/#silly)

`silly`（愚蠢）的身份验证提供者只适合于开发。它只是检查 HTTP 请求中是否存在 `Authorization` 头。它不检查报头的值。如果报头不存在，愚蠢的认证响应一个质询响应，回显访问被拒绝的领域、服务和范围。

以下值用于配置响应：

| 参数 | Required | 描述 |
| --- | --- | --- |
| `realm` | yes | 注册中心服务器进行身份验证的领域。 |
| `service` | yes | 正在验证的服务 |

### `token`[](https://ewhisper.cn/posts/48670/#token)

基于令牌的身份验证允许您将身份验证系统与注册表解耦。它是一种已建立的具有高度安全性的身份验证范例。

| 参数 | Required | 描述 |
| --- | --- | --- |
| `realm` | yes | 注册中心服务器进行身份验证的领域（realm）。 |
| `service` | yes | 正在验证的服务。 |
| `issuer` | yes | 令牌颁发者的名称。颁发者将此值插入到令牌中，因此它必须匹配为颁发者配置的值。 |
| `rootcertbundle` | yes | 根证书包的绝对路径。此包包含用于签署身份验证令牌的证书的公共部分。 |
| `autoredirect` | no | 当设置为 `true` 时，`realm`将自动被设置为使用请求的 Host 报头作为域和 `/auth/token/` 作为路径 |

有关基于令牌的身份验证配置的更多信息，请参见 [规范](https://docs.docker.com/registry/spec/auth/token/)。

### `htpasswd`[](https://ewhisper.cn/posts/48670/#htpasswd)

支持的 _htpasswd_ 身份验证允许您使用 [Apache htpasswd 文件](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) 配置基本身份验证。唯一支持的密码格式是 [`bcrypt`](http://en.wikipedia.org/wiki/Bcrypt)。其他哈希类型的条目将被忽略。`htpasswd` 文件只在启动时加载一次。如果文件无效，注册表将显示一个错误，并且不会启动。

> 警告：如果 `htpasswd`文件缺失，将使用默认用户和自动生成的密码创建和发放该文件。密码将打印到标准输出。

> 警告：只使用配置了 TLS 的 `htpasswd` 身份验证方案，因为基本身份验证将密码作为 HTTP 报头的一部分发送。

| 参数 | Required | 描述 |
| --- | --- | --- |
| `realm` | yes | 注册中心服务器进行身份验证的领域。 |
| `path` | yes | 启动时加载 `htpasswd` 文件的路径 |

## `middleware`[](https://ewhisper.cn/posts/48670/#middleware)

`middleware`结构是 **可选** 的。使用这个选项在指定的钩子点注入中间件。每个中间件必须实现与它所包装的对象相同的接口。例如，注册中心中间件必须实现 `distribution.Namespace` 接口，而 repository 中间件必须实现`distribution.Repository`, 和存储中间件必须实现`driver.StorageDriver`。

下面是 `cloudfront` 中间件（存储中间件）的一个配置示例：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br></pre></td><td><pre><code><span>middleware:</span><br>  <span>registry:</span><br>    <span>-</span> <span>name:</span> <span>ARegistryMiddleware</span><br>      <span>options:</span><br>        <span>foo:</span> <span>bar</span><br>  <span>repository:</span><br>    <span>-</span> <span>name:</span> <span>ARepositoryMiddleware</span><br>      <span>options:</span><br>        <span>foo:</span> <span>bar</span><br>  <span>storage:</span><br>    <span>-</span> <span>name:</span> <span>cloudfront</span><br>      <span>options:</span><br>        <span>baseurl:</span> <span>https://my.cloudfronted.domain.com/</span><br>        <span>privatekey:</span> <span>/path/to/pem</span><br>        <span>keypairid:</span> <span>cloudfrontkeypairid</span><br>        <span>duration:</span> <span>3000s</span><br>        <span>ipfilteredby:</span> <span>awsregion</span><br>        <span>awsregion:</span> <span>us-east-1,</span> <span>use-east-2</span><br>        <span>updatefrenquency:</span> <span>12h</span><br>        <span>iprangesurl:</span> <span>https://ip-ranges.amazonaws.com/ip-ranges.json</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

每个中间件条目都有 `name` 和`options`条目。`name`必须与中间件注册自身时使用的名称相对应。`options`字段是一个映射，它详细说明了初始化中间件所需的定制配置。它被视为一个`map[string]interface{}`。因此，它支持所需的任何有趣的结构，让中间件初始化函数来最好地决定如何处理选项的特定解释。

### `cloudfront`[](https://ewhisper.cn/posts/48670/#cloudfront)

| 参数 | Required | 描述 |
| --- | --- | --- |
| `baseurl` | yes | The `SCHEME://HOST[/PATH]` at which Cloudfront is served. |
| `privatekey` | yes | The private key for Cloudfront, provided by AWS. |
| `keypairid` | yes | The key pair ID provided by AWS. |
| `duration` | no | An integer and unit for the duration of the Cloudfront session. Valid time units are `ns`, `us` (or `µs`), `ms`, `s`, `m`, or `h`. For example, `3000s` is valid, but `3000 s` is not. If you do not specify a `duration` or you specify an integer without a time unit, the duration defaults to `20m` (20 minutes). |
| `ipfilteredby` | no | A string with the following value `none`, `aws` or `awsregion`. |
| `awsregion` | no | A comma separated string of AWS regions, only available when `ipfilteredby` is `awsregion`. For example, `us-east-1, us-west-2` |
| `updatefrenquency` | no | The frequency to update AWS IP regions, default: `12h` |
| `iprangesurl` | no | The URL contains the AWS IP ranges information, default: `https://ip-ranges.amazonaws.com/ip-ranges.json` |

`ipfilteredby`的取值包括：

| Value | Description |
| --- | --- |
| `none` | default, do not filter by IP |
| `aws` | IP from AWS goes to S3 directly |
| `awsregion` | IP from certain AWS regions goes to S3 directly, use together with `awsregion`. |

### `redirect`[](https://ewhisper.cn/posts/48670/#redirect-2)

您可以使用 `redirect` 存储中间件为 S3 存储驱动程序存储的层指定一个自定义 URL 到代理的位置。

| Parameter | Required | Description |
| --- | --- | --- |
| `baseurl` | yes | `SCHEME://HOST` at which layers are served. Can also contain port. For example, `https://example.com:5443`. |

## `reporting`[](https://ewhisper.cn/posts/48670/#reporting)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code><span>reporting:</span><br>  <span>bugsnag:</span><br>    <span>apikey:</span> <span>bugsnagapikey</span><br>    <span>releasestage:</span> <span>bugsnagreleasestage</span><br>    <span>endpoint:</span> <span>bugsnagendpoint</span><br>  <span>newrelic:</span><br>    <span>licensekey:</span> <span>newreliclicensekey</span><br>    <span>name:</span> <span>newrelicname</span><br>    <span>verbose:</span> <span>true</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`reporting`选项是可选的，用于配置错误和度量报告工具。目前只支持两种服务：

-   [Bugsnag](https://docs.docker.com/registry/configuration/#bugsnag)
-   [New Relic](https://docs.docker.com/registry/configuration/#new-relic)

有效的配置可以同时包含这两种配置。

### `bugsnag`[](https://ewhisper.cn/posts/48670/#bugsnag)

| Parameter | Required | Description |
| --- | --- | --- |
| `apikey` | yes | The API Key provided by Bugsnag. |
| `releasestage` | no | Tracks where the registry is deployed, using a string like `production`, `staging`, or `development`. |
| `endpoint` | no | The enterprise Bugsnag endpoint. |

### `newrelic`[](https://ewhisper.cn/posts/48670/#newrelic)

| Parameter | Required | Description |
| --- | --- | --- |
| `licensekey` | yes | License key provided by New Relic. |
| `name` | no | New Relic application name. |
| `verbose` | no | Set to `true` to enable New Relic debugging output on `stdout`. |

## `http`[](https://ewhisper.cn/posts/48670/#http)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br></pre></td><td><pre><code><span>http:</span><br>  <span>addr:</span> <span>localhost:5000</span><br>  <span>net:</span> <span>tcp</span><br>  <span>prefix:</span> <span>/my/nested/registry/</span><br>  <span>host:</span> <span>https://myregistryaddress.org:5000</span><br>  <span>secret:</span> <span>asecretforlocaldevelopment</span><br>  <span>relativeurls:</span> <span>false</span><br>  <span>draintimeout:</span> <span>60s</span><br>  <span>tls:</span><br>    <span>certificate:</span> <span>/path/to/x509/public</span><br>    <span>key:</span> <span>/path/to/x509/private</span><br>    <span>clientcas:</span><br>      <span>-</span> <span>/path/to/ca.pem</span><br>      <span>-</span> <span>/path/to/another/ca.pem</span><br>    <span>minimumtls:</span> <span>tls1.2</span><br>    <span>ciphersuites:</span><br>      <span>-</span> <span>TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384</span><br>      <span>-</span> <span>TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384</span><br>    <span>letsencrypt:</span><br>      <span>cachefile:</span> <span>/path/to/cache-file</span><br>      <span>email:</span> <span>emailused@letsencrypt.com</span><br>      <span>hosts:</span> [<span>myregistryaddress.org</span>]<br>  <span>debug:</span><br>    <span>addr:</span> <span>localhost:5001</span><br>  <span>headers:</span><br>    <span>X-Content-Type-Options:</span> [<span>nosniff</span>]<br>  <span>http2:</span><br>    <span>disabled:</span> <span>false</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`http`选项详细说明了承载注册中心的 http 服务器的配置。

| Parameter | Required | Description |
| --- | --- | --- |
| `addr` | yes | The address for which the server should accept connections. The form depends on a network type (see the `net` option). Use `HOST:PORT` for TCP and `FILE` for a UNIX socket. 服务器应该接受连接的地址。表单取决于网络类型（请参阅 `net` 选项）。使用`HOST:PORT` for tcp, 使用`FILE` for UNIX 套接字。 |
| `net` | no | 用于创建监听套接字的网络。已知的网络有 `unix` and `tcp`. |
| `prefix` | no | 如果服务器不在根路径上运行，则将其设置为前缀的值。根路径是 `v2` 之前的部分。它需要前斜杠和后斜杠，例如示例中的`/path/`。 |
| `host` | no | 注册表外部可达地址的完全限定 URL。如果存在，则在创建生成的 url 时使用它。否则，这些 url 派生自客户端请求。 |
| `secret` | no | 一种用于签名状态的随机数据，可以存储在客户端以防止篡改。对于生产环境，您应该使用加密安全的随机生成器生成一段随机数据。如果省略该秘密，注册表将在启动时自动生成一个秘密。**如果您在负载均衡器后面构建一个注册表集群，您必须确保所有注册表的秘密是相同的。** |
| `relativeurls` | no | 如果为`true`，注册表将在 Location 头中返回相对 url。客户端负责解析正确的 URL。此选项不兼容 Docker 1.7 及更早版本。 |
| `draintimeout` | no | 注册表收到 SIGTERM 信号后，在关闭之前等待 HTTP 连接耗尽的时间 |

### `tls`[](https://ewhisper.cn/posts/48670/#tls)

`http`中的 `tls` 结构是可选的。用于为服务器配置 TLS。如果您已经有一个 web 服务器运行在与注册表相同的主机上，您可能更喜欢在该 web 服务器上配置 TLS 和到注册表服务器的代理连接。

| Parameter | Required | Description |
| --- | --- | --- |
| `certificate` | yes | x509 证书文件的绝对路径。 |
| `key` | yes | x509 私钥文件的绝对路径。 |
| `clientcas` | no | x509 CA 文件的绝对路径的数组。 |
| `minimumtls` | no | TLS 最小版本 (tls1.0, tls1.1, tls1.2, tls1.3)。默认为 tls1.2 |
| `ciphersuites` | no | 允许的密码套件。请查看下面允许的值和默认值。 |

可用密码套件：

-   TLS\_RSA\_WITH\_RC4\_128\_SHA
-   TLS\_RSA\_WITH\_3DES\_EDE\_CBC\_SHA
-   TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA
-   TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA
-   TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA256
-   TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256
-   TLS\_RSA\_WITH\_AES\_256\_GCM\_SHA384
-   TLS\_ECDHE\_ECDSA\_WITH\_RC4\_128\_SHA
-   TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CBC\_SHA
-   TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_CBC\_SHA
-   TLS\_ECDHE\_RSA\_WITH\_RC4\_128\_SHA
-   TLS\_ECDHE\_RSA\_WITH\_3DES\_EDE\_CBC\_SHA
-   TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA
-   TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA
-   TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CBC\_SHA256
-   TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA256
-   TLS\_ECDHE\_RSA\_WITH\_AES\_128\_GCM\_SHA256
-   TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256
-   TLS\_ECDHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384
-   TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384
-   TLS\_ECDHE\_RSA\_WITH\_CHACHA20\_POLY1305\_SHA256
-   TLS\_ECDHE\_ECDSA\_WITH\_CHACHA20\_POLY1305\_SHA256
-   TLS\_AES\_128\_GCM\_SHA256
-   TLS\_AES\_256\_GCM\_SHA384
-   TLS\_CHACHA20\_POLY1305\_SHA256

默认密码套件：

-   TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384
-   TLS\_ECDHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384
-   TLS\_ECDHE\_ECDSA\_WITH\_CHACHA20\_POLY1305\_SHA256
-   TLS\_ECDHE\_RSA\_WITH\_CHACHA20\_POLY1305\_SHA256
-   TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256
-   TLS\_ECDHE\_RSA\_WITH\_AES\_128\_GCM\_SHA256
-   TLS\_AES\_128\_GCM\_SHA256
-   TLS\_CHACHA20\_POLY1305\_SHA256
-   TLS\_AES\_256\_GCM\_SHA384

### `letsencrypt`[](https://ewhisper.cn/posts/48670/#letsencrypt)

`tls`中的 `letsencrypt` 结构是可选的。使用此配置由“[Let’s Encrypt](https://letsencrypt.org/how-it-works/)”提供 TLS 证书。

> 注意：当使用 Let’s Encrypt 时，请确保在端口 `443` 上可以访问向外的地址。注册表默认监听端口 5000。如果您将注册表作为容器运行，请考虑在 `docker run` 命令中添加标志`-p 443:5000`，或者在云配置中使用类似的设置。您还应该将`hosts` 选项设置为对该注册表有效的主机名列表，以避免由于恶意客户机使用伪 SNI 主机名连接而试图获取随机主机名的证书。

| Parameter | Required | Description |
| --- | --- | --- |
| `cachefile` | yes | Let’s Encrypt agent 可以缓存数据的文件的绝对路径。 |
| `email` | yes | 注册 Let 's Encrypt 时使用的电子邮件地址。 |
| `hosts` | no | Let’s Encrypt 证书允许的主机名。 |

### `debug`[](https://ewhisper.cn/posts/48670/#debug)

`debug`选项是 **可选** 的。使用它可以配置调试服务器，这有助于诊断问题。调试端点可用于监视注册表指标和运行状况，以及分析。敏感信息可以通过调试端点获得。请确保在生产环境中锁定了对调试端点的访问。

`debug`部分接受一个必需的 `addr` 参数，它指定调试服务器应该在其上接受连接的`HOST:PORT`。

## `prometheus`[](https://ewhisper.cn/posts/48670/#prometheus)

`prometheus`选项定义是否启用 prometheus 度量标准，以及访问度量标准的路径。

| Parameter | Required | Description |
| --- | --- | --- |
| `enabled` | no | 设置为`true`，开启普罗米修斯服务器 |
| `path` | no | 访问度量的路径，默认情况下为`/metrics` |

访问指标的 url 是 `HOST:PORT/path`，其中`HOST:PORT` 是在 `debug` 下的 `addr` 中定义的。

`headers`选项是 **可选** 的。使用它来指定 HTTP 服务器应该在响应中包含的头。这可以用于安全头，如`Strict-Transport-Security`。

`headers`选项应该包含每个报头要包含的选项，其中参数名是报头的名称，参数值是报头有效载荷值的列表。

建议包含`X-Content-Type-Options: [nosniff]`，这样，如果浏览器被引导从注册表加载页面，就不会将内容解释为 HTML。这个头文件包含在示例配置文件中。

### `http2`[](https://ewhisper.cn/posts/48670/#http2)

`http`中的 `http2` 结构是可选的。使用它来控制注册表的 http2 设置。

| Parameter | Required | Description |
| --- | --- | --- |
| `disabled` | no | If `true`, then `http2` support is disabled. |

## `notifications`[](https://ewhisper.cn/posts/48670/#notifications)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br></pre></td><td><pre><code><span>notifications:</span><br>  <span>events:</span><br>    <span>includereferences:</span> <span>true</span><br>  <span>endpoints:</span><br>    <span>-</span> <span>name:</span> <span>alistener</span><br>      <span>disabled:</span> <span>false</span><br>      <span>url:</span> <span>https://my.listener.com/event</span><br>      <span>headers:</span> <span>&lt;http.Header&gt;</span><br>      <span>timeout:</span> <span>1s</span><br>      <span>threshold:</span> <span>10</span><br>      <span>backoff:</span> <span>1s</span><br>      <span>ignoredmediatypes:</span><br>        <span>-</span> <span>application/octet-stream</span><br>      <span>ignore:</span><br>        <span>mediatypes:</span><br>           <span>-</span> <span>application/octet-stream</span><br>        <span>actions:</span><br>           <span>-</span> <span>pull</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`notifications`选项是可选的，目前可能包含单个选项 `endpoints`。

### `endpoints`[](https://ewhisper.cn/posts/48670/#endpoints)

`endpoints`结构包含可以接受事件通知的命名服务 (url) 列表。

| Parameter | Required | Description |
| --- | --- | --- |
| `name` | yes | 服务的人类可读的名称。 |
| `disabled` | no | 如果为`true`，则禁用该服务的通知。 |
| `url` | yes | 事件应该发布到的 URL。 |
| `headers` | yes | 要添加到每个请求的静态报头列表。每个报头的名称是报头下面的一个键，每个值是该报头名称的有效负载列表。值必须始终是列表。 |
| `timeout` | yes | HTTP 超时的值。一个正整数和一个指示时间单位的可选后缀，可以是 `ns`, `us`, `ms`, `s`, `m`, or `h`. 如果省略时间单位，则使用`ns`。 |
| `threshold` | yes | 一个整数，指定在退回失败之前等待多长时间。 |
| `backoff` | yes | 出现故障后，系统在重试之前退回的时间。一个正整数和一个指示时间单位的可选后缀，可以是 `ns`, `us`, `ms`, `s`, `m`, or `h`. 如果省略时间单位，则使用`ns`。 |
| `ignoredmediatypes` | no | 要忽略的目标媒体类型的列表。具有这些目标媒体类型的事件不会发布到端点 |
| `ignore` | no | 具有这些媒体类型或操作的事件不会发布到端点。 |

#### `ignore`[](https://ewhisper.cn/posts/48670/#ignore)

| Parameter | Required | Description |
| --- | --- | --- |
| `mediatypes` | no | 要忽略的目标媒体类型的列表。具有这些目标媒体类型的事件不会发布到端点。 |
| `actions` | no | 可以忽略的行动列表。带有这些操作的事件不会发布到端点。 |

### `events`[](https://ewhisper.cn/posts/48670/#events)

`events`结构配置事件通知中提供的信息。

| Parameter | Required | Description |
| --- | --- | --- |
| `includereferences` | no | 如果为真，则在 manifest 事件中包含引用信息。 |

## `redis`[](https://ewhisper.cn/posts/48670/#redis)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><code><span>redis:</span><br>  <span>addr:</span> <span>localhost:6379</span><br>  <span>password:</span> <span>asecret</span><br>  <span>db:</span> <span>0</span><br>  <span>dialtimeout:</span> <span>10ms</span><br>  <span>readtimeout:</span> <span>10ms</span><br>  <span>writetimeout:</span> <span>10ms</span><br>  <span>pool:</span><br>    <span>maxidle:</span> <span>16</span><br>    <span>maxactive:</span> <span>64</span><br>    <span>idletimeout:</span> <span>300s</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

声明用于构建 `redis` 连接的参数。注册表实例可以在多个应用程序中使用 Redis 实例。目前，它缓存关于不可变 blobs 的信息。大多数 `redis` 选项控制注册表如何连接到 `redis` 实例。您可以使用 [`pool`](https://docs.docker.com/registry/configuration/#pool) 分段来控制池的行为。

你应该配置 Redis 为**allkeys-lru** eviction 策略，因为注册表不会为 key 设置过期值。

| Parameter | Required | Description |
| --- | --- | --- |
| `addr` | yes | Redis 实例的地址（主机和端口）。 |
| `password` | no | 用于验证 Redis 实例的密码。 |
| `db` | no | 要用于每个连接的数据库的名称。 |
| `dialtimeout` | no | 连接 Redis 实例的超时时间。 |
| `readtimeout` | no | 从 Redis 实例读取的超时时间。 |
| `writetimeout` | no | 写入 Redis 实例的超时时间。 |

### `pool`[](https://ewhisper.cn/posts/48670/#pool)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>pool:</span><br>  <span>maxidle:</span> <span>16</span><br>  <span>maxactive:</span> <span>64</span><br>  <span>idletimeout:</span> <span>300s</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

使用这些设置配置 Redis 连接池的行为。

| Parameter | Required | Description |
| --- | --- | --- |
| `maxidle` | no | 池中最大空闲连接数。 |
| `maxactive` | no | 在阻塞连接请求之前可以打开的最大连接数。 |
| `idletimeout` | no | 关闭不活动连接前需要等待多长时间。 |

## `health`[](https://ewhisper.cn/posts/48670/#health)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br></pre></td><td><pre><code><span>health:</span><br>  <span>storagedriver:</span><br>    <span>enabled:</span> <span>true</span><br>    <span>interval:</span> <span>10s</span><br>    <span>threshold:</span> <span>3</span><br>  <span>file:</span><br>    <span>-</span> <span>file:</span> <span>/path/to/checked/file</span><br>      <span>interval:</span> <span>10s</span><br>  <span>http:</span><br>    <span>-</span> <span>uri:</span> <span>http://server.to.check/must/return/200</span><br>      <span>headers:</span><br>        <span>Authorization:</span> [<span>Basic</span> <span>QWxhZGRpbjpvcGVuIHNlc2FtZQ==</span>]<br>      <span>statuscode:</span> <span>200</span><br>      <span>timeout:</span> <span>3s</span><br>      <span>interval:</span> <span>10s</span><br>      <span>threshold:</span> <span>3</span><br>  <span>tcp:</span><br>    <span>-</span> <span>addr:</span> <span>redis-server.domain.com:6379</span><br>      <span>timeout:</span> <span>3s</span><br>      <span>interval:</span> <span>10s</span><br>      <span>threshold:</span> <span>3</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`health`选项是 **可选** 的，包含存储驱动程序后端存储上定期运行状况检查的首选项，以及本地文件、HTTP uri 和 / 或 TCP 服务器上的可选定期检查。如果启用了调试 HTTP 服务器（参见 HTTP 部分），运行状况检查的结果可以在调试 HTTP 服务器的 `/debug/health` 状况端点获得。

### `storagedriver`[](https://ewhisper.cn/posts/48670/#storagedriver)

`storagedriver`结构包含对已配置的存储驱动程序的后端存储进行健康检查的选项。当 `enabled` 设置为 `true` 时，健康检查是激活的。

| Parameter | Required | Description |
| --- | --- | --- |
| `enabled` | yes | 设置为 `true` 表示启用存储驱动程序健康检查，设置为 `false` 表示禁用存储驱动程序健康检查。 |
| `interval` | no | 重复存储驱动程序运行状况检查之间的等待时间。一个正整数和一个指示时间单位的可选后缀。后缀是 `ns`, `us`, `ms`, `s`, `m`, or `h`. Defaults to `10s` if the value is omitted. 如果您指定了一个值但省略了后缀，则该值将被解释为纳秒数。 |
| `threshold` | no | 一个正整数，表示在将状态标记为不健康之前，检查必须失败的次数。如果未指定，单个故障将将状态标记为不健康。 |

### `file`[](https://ewhisper.cn/posts/48670/#file)

`file`结构包括一个定期检查文件是否存在的路径列表。如果在指定路径上存在文件，则健康检查将失败。您可以使用这种机制通过创建一个文件使注册表停止 rotation 。

| Parameter | Required | Description |
| --- | --- | --- |
| `file` | yes | 检查文件是否存在的路径。 |
| `interval` | no | How long to wait before repeating the check. A positive integer and an optional suffix indicating the unit of time. The suffix is one of `ns`, `us`, `ms`, `s`, `m`, or `h`. Defaults to `10s` if the value is omitted. If you specify a value but omit the suffix, the value is interpreted as a number of nanoseconds. |

### `http`[](https://ewhisper.cn/posts/48670/#http-2)

`http`结构包括一个 http uri 列表，可以通过 `HEAD` 请求定期检查。如果 HEAD 请求没有完成或返回一个意外的状态码，则健康检查将失败。

| Parameter | Required | Description |
| --- | --- | --- |
| `uri` | yes | The URI to check. |
| `headers` | no | Static headers to add to each request. Each header’s name is a key beneath `headers`, and each value is a list of payloads for that header name. Values must always be lists. |
| `statuscode` | no | The expected status code from the HTTP URI. Defaults to `200`. |
| `timeout` | no | How long to wait before timing out the HTTP request. A positive integer and an optional suffix indicating the unit of time. The suffix is one of `ns`, `us`, `ms`, `s`, `m`, or `h`. If you specify a value but omit the suffix, the value is interpreted as a number of nanoseconds. |
| `interval` | no | How long to wait before repeating the check. A positive integer and an optional suffix indicating the unit of time. The suffix is one of `ns`, `us`, `ms`, `s`, `m`, or `h`. Defaults to `10s` if the value is omitted. If you specify a value but omit the suffix, the value is interpreted as a number of nanoseconds. |
| `threshold` | no | The number of times the check must fail before the state is marked as unhealthy. If this field is not specified, a single failure marks the state as unhealthy. |

### `tcp`[](https://ewhisper.cn/posts/48670/#tcp)

`tcp`结构包括一个 tcp 地址列表，可以使用 tcp 连接尝试定期检查这些地址。地址必须包含端口号。如果连接失败，则健康检查失败。

| Parameter | Required | Description |
| --- | --- | --- |
| `addr` | yes | The TCP address and port to connect to. |
| `timeout` | no | How long to wait before timing out the TCP connection. A positive integer and an optional suffix indicating the unit of time. The suffix is one of `ns`, `us`, `ms`, `s`, `m`, or `h`. If you specify a value but omit the suffix, the value is interpreted as a number of nanoseconds. |
| `interval` | no | How long to wait between repetitions of the check. A positive integer and an optional suffix indicating the unit of time. The suffix is one of `ns`, `us`, `ms`, `s`, `m`, or `h`. Defaults to `10s` if the value is omitted. If you specify a value but omit the suffix, the value is interpreted as a number of nanoseconds. |
| `threshold` | no | The number of times the check must fail before the state is marked as unhealthy. If this field is not specified, a single failure marks the state as unhealthy. |

## `proxy`[](https://ewhisper.cn/posts/48670/#proxy)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>proxy:</span><br>  <span>remoteurl:</span> <span>https://registry-1.docker.io</span><br>  <span>username:</span> [<span>username</span>]<br>  <span>password:</span> [<span>password</span>]<br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

`proxy`结构允许注册表被配置为 Docker Hub 的一个 `pull-through` 缓存。更多信息见 [mirror](https://github.com/docker/docker.github.io/tree/master/registry/recipes/mirror.md)。不支持 push 到配置为 pull-through 缓存的注册表。

| Parameter | Required | Description |
| --- | --- | --- |
| `remoteurl` | yes | Docker Hub 上存储库的 URL。 |
| `username` | no | 注册到 Docker Hub 的可以访问存储库的用户名。 |
| `password` | no | 使用“`username`”中指定的用户名验证到 Docker Hub 的密码。 |

要启用 pulling 私有存储库（如 batman/robin)，请指定用户名（如 batman) 和该用户名的密码。

> 注意：这些私有存储库存储在代理缓存的存储中。采取适当的措施来保护对代理缓存的访问。

## `compatibility`[](https://ewhisper.cn/posts/48670/#compatibility)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code>compatibility:<br>  schema1:<br>    signingkeyfile: /etc/registry/key.json<br>    enabled: true<br></code><p><i></i>bash</p></pre></td></tr></tbody></table>

使用 `compatibility` 结构来配置对旧的和已弃用的特性的处理。每个小节定义了具有可配置行为的这种特性。

### `schema1`[](https://ewhisper.cn/posts/48670/#schema1)

| Parameter | Required | Description |
| --- | --- | --- |
| `signingkeyfile` | no | The signing private key used to add signatures to `schema1` manifests. If no signing key is provided, a new ECDSA key is generated when the registry starts. |
| `enabled` | no | If this is not set to true, `schema1` manifests cannot be pushed. |

## `validation`[](https://ewhisper.cn/posts/48670/#validation)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code>validation:<br>  manifests:<br>    urls:<br>      allow:<br>        - ^https?://([^/]+\.)*example\.com/<br>      deny:<br>        - ^https?://www\.example\.com/<br></code><p><i></i>bash</p></pre></td></tr></tbody></table>

### `disabled`[](https://ewhisper.cn/posts/48670/#disabled)

`disabled`标志禁用了 `validation` 部分中的其他选项。默认情况下是启用的。This option deprecates the `enabled` flag.

### `manifests`[](https://ewhisper.cn/posts/48670/#manifests)

使用 `manifests` 小节配置清单的验证。如果 `disabled` 为`false`，则验证不允许任何操作。

#### `urls`[](https://ewhisper.cn/posts/48670/#urls)

`allow` and `deny` 选项都是一个 [正则表达式](https://godoc.org/regexp/syntax) 列表，用于限制 pushed manifests 中的 url。

如果未设置`allow`，则推送包含 url 的清单将失败。

如果设置了 `allow`，那么只有当所有的 url 都匹配`allow` 正则表达式中的一个，并且有以下一个匹配时，才会成功推送清单：

1.  `deny` 未设置
2.  `deny`已设置，但是清单中没有 url 匹配任何 `deny` 正则表达式。

## 示例：开发配置[](https://ewhisper.cn/posts/48670/#%E7%A4%BA%E4%BE%8B%EF%BC%9A%E5%BC%80%E5%8F%91%E9%85%8D%E7%BD%AE)

你可以使用这个简单的例子来进行本地开发：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><code><span>version:</span> <span>0.1</span><br><span>log:</span><br>  <span>level:</span> <span>debug</span><br><span>storage:</span><br>    <span>filesystem:</span><br>        <span>rootdirectory:</span> <span>/var/lib/registry</span><br><span>http:</span><br>    <span>addr:</span> <span>localhost:5000</span><br>    <span>secret:</span> <span>asecretforlocaldevelopment</span><br>    <span>debug:</span><br>        <span>addr:</span> <span>localhost:5001</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

这个示例将注册表实例配置为在端口 `5000` 上运行，绑定到 `localhost`，并启用`debug` 服务器。注册表数据存放在“`/var/lib/registry`”目录下。日志记录被设置为 `debug` 模式，这是最详细的。

参见另一个简单配置 [config-example.yml](https://github.com/docker/distribution/blob/master/cmd/registry/config-example.yml) 。这两个例子通常对本地开发是有用的。

## 示例：中间件配置[](https://ewhisper.cn/posts/48670/#%E7%A4%BA%E4%BE%8B%EF%BC%9A%E4%B8%AD%E9%97%B4%E4%BB%B6%E9%85%8D%E7%BD%AE)

本示例将 [Amazon Cloudfront](http://aws.amazon.com/cloudfront/) 配置为注册中心中的存储中间件。中间件允许注册中心通过内容传递网络 (CDN) 为各层提供服务。这减少了对存储层的请求。

Cloudfront 需要 S3 存储驱动程序。

下面是用 YAML 表示的配置：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code><span>middleware:</span><br>  <span>storage:</span><br>  <span>-</span> <span>name:</span> <span>cloudfront</span><br>    <span>disabled:</span> <span>false</span><br>    <span>options:</span><br>      <span>baseurl:</span> <span>http://d111111abcdef8.cloudfront.net</span><br>      <span>privatekey:</span> <span>/path/to/asecret.pem</span><br>      <span>keypairid:</span> <span>asecret</span><br>      <span>duration:</span> <span>60s</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

有关配置选项的更多信息，请参阅 [Cloudfront](https://docs.docker.com/registry/configuration/#cloudfront) 的配置参考。

> 注意：Cloudfront 密钥与其他 AWS 密钥分开存在。有关更多信息，请参阅 [AWS 凭据文档](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html)。