---
title: "57-bits の仮想メモリアドレス空間と新機能 UAI が実装される将来の AMD プロセッサ"
date: 2022-03-12T08:13:30+09:00
draft: false
categories: [ "Software", "AMD", "CPU" ]
tags: [ "Linux_Kernel", "Zen_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の [Bharata B Rao](https://in.linkedin.com/in/bharata-b-rao-58123851) 氏より linux-mm (memory management) メーリングリストに、近く登場する AMDプロセッサでサポートされる AMD UAI (Upper Address Ignore) をメモリアドレスへのメタデータ、タグの付与に活用するパッチが投稿されている。  
AMD UAI は仮想アドレスの上位 7-bit (Bit_63:57) に対するチェックを抑制する機能であり、ソフトウェアはその分をメタデータ、タグの格納に活用することができる。  

* [[RFC PATCH v0 0/6] x86/AMD: Userspace address tagging](https://lore.kernel.org/linux-mm/2b26fc5b-d709-f2e1-0c8f-a6a548008216@intel.com/T/)

UAI のサポートフラグ `CPUID.(EAX=0x8000_0021, ECX=0x0):EAX[bit 7]` から確認可能としている。  

 > 		@@ -42,6 +42,7 @@ static const struct cpuid_bit cpuid_bits[] = {
 > 		 	{ X86_FEATURE_CPB,		CPUID_EDX,  9, 0x80000007, 0 },
 > 		 	{ X86_FEATURE_PROC_FEEDBACK,    CPUID_EDX, 11, 0x80000007, 0 },
 > 		 	{ X86_FEATURE_MBA,		CPUID_EBX,  6, 0x80000008, 0 },
 > 		+	{ X86_FEATURE_UAI,		CPUID_EAX,  7, 0x80000021, 0 },
 >
 > {{< quote >}} [[RFC PATCH v0 2/6] x86/cpufeatures: Add Upper Address Ignore(UAI) as CPU feature - Bharata B Rao](https://lore.kernel.org/linux-mm/20220310111545.10852-3-bharata@amd.com/) {{< /quote >}}

パッチコメントにて、現 AMDプロセッサは最大 57-bits の仮想 (virtual/logical) アドレスサイズをサポートするとしているが、*AMD Zen 3 アーキテクチャ* は最大 48-bit であるため、次世代の AMDプロセッサ、*AMD Zen 4 アーキテクチャ* を指しているのではないかと思われる。  

*AMD Zen 1/2/3* では物理アドレス空間、仮想アドレス空間ともに最大 48-bit のサポートとなっている。  
*AMD Zen 1/2/3* ではメモリ暗号化機能、SME (Secure Memory Encryption ) か SEV (Secure Encrypted Virtualiza) が有効な場合、使用可能な物理アドレス空間が減る。  
減るアドレスサイズは `CPUID (Leaf: 0x8000_001F) -> EBX Bit_11:6` から確認できる。  
自分が確認できる限りだと、減るアドレスサイズは *Ryzen 5 2600 (Zen 1)* では 5-bits となっていた。*Ryzen 5 5600G (Zen 3)* では 0 だっため、物理アドレスサイズは減らない。  

 > 		Currently the maximum logical address size for AMD processors in
 > 		64 bit mode is 57 bits. This means that the remaining 7 upper bits
 > 		[63:57] are available for software use. With UAI feature turned ON,
 > 		the processor ignores these upper bits when performing canonical
 > 		check on these addresses.
 > 		
 > 		Add UAI as a CPU feature.
 >
 > {{< quote >}} [[RFC PATCH v0 0/6] x86/AMD: Userspace address tagging](https://lore.kernel.org/linux-mm/2b26fc5b-d709-f2e1-0c8f-a6a548008216@intel.com/T/#e1b9caa0c700839bc9238a3161ddc5b757062d077) {{< /quote >}}

## AMD UAI が抱える問題 {#problem}

メモリアドレスの一部をチェックの対象から外し、メタデータあるいはタグとして活用する機能は他でも策定と実装が進められており、Intel LAM (Linear-address Masking)、Arm TBI (Top-byte Ignore) と Arm MTE (Memory Tagging Extension) がある。  
用途として、Arm MTE の例ではタグの一致を確認することで、メモリ安全性に関わる 2つのエラー、解放後メモリの使用 (Use-After-Free) と領域外アクセス (out-of-bounds) を検知可能としている。  

それら機能の主な差異としては、メタデータのサイズと位置が挙げられ、  
Intel LAM は LAM_U48 の場合 Bit_62:48 (15-bits) を、LAM_U57 の場合は Bit_62:57 (6-bits) をメタデータとして用いる。  
Arm TBI は Bit_63:56 (8-bits) を無視し、その内の一部、Bit_59:56 (4-bits) を Arm MTE でメタデータとして用いる。  
AMD UAI は Bit_63:57 (7-bits) を無視するため、メタデータのサイズと位置が一致するものはなく、それぞれで異なる。  
無視されるビットすべてをメタデータとして活用しなければ、機能間で仕様をある程度一致させることは可能かもしれないが。  

AMD UAI の問題点には Intel LAM と互換性が無いことだけでなく、Bit_63 も無視してしまうことが指摘されている。  
Bit_63 は Linux Kernel において通常、そのアドレスが User か Kernel かを判別するために使用されており、Bit_63:57 を無視する機能である AMD UAI は、ユーザースペースからそれを変更することを可能としてしまう。  
Intel LAM ではその点が考慮されており、Bit_63 を無視しない仕様となっている。  
また AMD UAI はコンテキストスイッチを発生され、それによって性能が低下することが懸念されている。  

 > 		Is that really allowing bit 63 to be used?
 > 		That is normally the user-kernel bit.
 > 		I can't help feeling that will just badly break things.
 >
 > {{< quote >}} [RE: [RFC PATCH v0 0/6] x86/AMD: Userspace address tagging - David Laight](https://lore.kernel.org/linux-mm/699fb763ac054833bc8c29c9814c63b2@AcuMS.aculab.com/) {{< /quote >}}

サポートするソフトウェア側から見たとき、AMD UAI より Intel LAM が明らかに優れているという意見も出されている。  

 > 		UAI and LAM are incompatible from a userspace perspective.  Since LAM is pretty clearly superior [0], it seems like a better long term outcome would be for programs that want tag bits to target LAM and for AMD to support LAM if there is demand.  For that matter, do we actually expect any userspace to want to support UAI?  (Are there existing too-clever sandboxes that would be broken by enabling UAI?)
 >
 > {{< quote >}} [Re: [RFC PATCH v0 0/6] x86/AMD: Userspace address tagging - Andy Lutomirski](https://lore.kernel.org/linux-mm/6a5076ad-405e-4e5e-af55-fe2a6b01467d@www.fastmail.com/) {{< /quote >}}

パッチを提出した AMD の [Bharata B Rao](https://in.linkedin.com/in/bharata-b-rao-58123851) 氏は、ハードウェアチームと話し合い、解決への最善策を探るとコメントしている。  
仮に AMD UAI の仕様が修正され、Intel LAM と同様のサイズ、位置になる場合、すでに実装されているプロセッサはマイクロコードによる修正等が必要になると思われる。  

 > 		We are discussing these aspects with the hardware team to check the best
 > 		possible path forward.
 >
 > {{< quote >}} [Re: [RFC PATCH v0 0/6] x86/AMD: Userspace address tagging - Bharata B Rao](https://lore.kernel.org/linux-mm/c4176dc5-af4d-243d-fce9-d7f45e79246a@amd.com/) {{< /quote >}}

{{< ref >}}
* [Enhanced security through Memory Tagging Extension - Architectures and Processors blog - Arm Community blogs - Arm Community](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/enhanced-security-through-mte)
* [セキュリティ・レポート｜株式会社ＦＦＲＩセキュリティ-サイバーセキュリティ、エンドポイントセキュリティ](https://www.ffri.jp/research/index.htm)
    * [recent_advances_vuln_mitig_jp.pdf](https://www.ffri.jp/assets/files/research/research_papers/recent_advances_vuln_mitig_jp.pdf)
* [memory-tagging-extension.rst « arm64 « Documentation - kernel/git/stable/linux.git - Linux kernel stable tree](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Documentation/arm64/memory-tagging-extension.rst?h=v5.16.14)
* [[RFC 0/9] Linear Address Masking enabling](https://lore.kernel.org/linux-mm/20210205151631.43511-2-kirill.shutemov@linux.intel.com/T/#u)
* [Home · Wiki · x86 psABIs / x86-64 psABI · GitLab](https://gitlab.com/x86-psABIs/x86-64-ABI/-/wikis/home)
* [Intel® Architecture Instruction Set Extensions Programming Reference](https://cdrdv2.intel.com/v1/dl/getContent/671368)
{{< /ref >}}
